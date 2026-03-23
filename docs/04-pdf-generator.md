# 📄 4. PDF Invoice Generator (Scalable Printing)

การสร้าง PDF จาก HTML/CSS เป็นวิธีที่นิยมที่สุดเพราะจัด Layout ได้ง่ายเหมือนทำเว็บ แต่ท้าทายเรื่องประสิทธิภาพเมื่อต้องรองรับการออกใบเสร็จพร้อมกันหลักพันใบ

## 🏗️ สถาปัตยกรรมระดับลึก (In-depth Architecture)

### 1. Headless Browser Rendering
ใช้ **Puppeteer** หรือ **Playwright** ในการเปิด Browser จำลอง (Headless) เพื่อ Render หน้าเว็บ HTML ให้กลายเป็นไฟล์ PDF 
* **Flexibility:** รองรับ CSS3, Web Fonts (ภาษาไทย) และ Flexbox/Grid ทำให้ดีไซน์ใบเสร็จได้สวยงาม
* **Complexity:** Browser กินทรัพยากรสูง จึงไม่ควรสั่งรันสดๆ ใน Main API Thread

### 2. Worker Queue Pattern
เพื่อป้องกันไม่ให้ API ล่มเมื่อมีการสั่งออกใบเสร็จจำนวนมาก เราจะใช้ระบบคิว (Queue) เข้ามาจัดการ
* **Producer (API):** รับ Request เก็บข้อมูลลง Database และโยนงาน (Job) เข้าคิว (เช่น **BullMQ** บน Redis)
* **Consumer (Worker):** ดึงงานจากคิวไปรัน Puppeteer เพื่อสร้าง PDF และส่งอีเมลหรืออัปโหลดขึ้น S3
* **Isolation:** หาก Worker ตัวหนึ่งค้างหรือล่ม จะไม่ส่งผลกระทบต่อ API หลักที่รับออเดอร์ลูกค้า

---

## 🧪 Lab: Scalable PDF Worker with BullMQ

Lab นี้จะจำลองการแยกส่วน API และ Worker เพื่อสร้าง PDF แบบไม่ขัดจังหวะการทำงานของระบบหลัก

### 🟩 Step 1: API (Producer) - ส่งงานเข้าคิว
```javascript
const { Queue } = require('bullmq');
const invoiceQueue = new Queue('invoice-pdf-generation');

app.post('/api/generate-invoice', async (req, res) => {
  const { orderId, customerName, amount } = req.body;
  
  // โยนงานเข้า Queue และตอบกลับลูกค้าทันที (ไม่ต้องรอสร้าง PDF เสร็จ)
  await invoiceQueue.add('generate', { orderId, customerName, amount });
  
  res.json({ message: "Invoice is being processed. You will receive it via email." });
});
```
🟧 Step 2: Worker (Consumer) - ประมวลผลสร้าง PDF
```Javascript
const { Worker } = require('bullmq');
const puppeteer = require('puppeteer');

const worker = new Worker('invoice-pdf-generation', async (job) => {
  console.log(`Processing Invoice for Order: ${job.data.orderId}`);
  
  const browser = await puppeteer.launch({ args: ['--no-sandbox'] });
  const page = await browser.newPage();
  
  // กำหนด HTML Template พร้อมรองรับภาษาไทย
  const htmlContent = `
    <html>
      <style> @font-face { font-family: 'Sarabun'; src: url('path/to/font.ttf'); } </style>
      <body style="font-family: 'Sarabun'">
        <h1>ใบเสร็จรับเงิน #${job.data.orderId}</h1>
        <p>คุณ: ${job.data.customerName}</p>
        <p>ยอดชำระ: ${job.data.amount} บาท</p>
      </body>
    </html>
  `;

  await page.setContent(htmlContent);
  const pdfBuffer = await page.pdf({ format: 'A4', printBackground: true });
  
  // ต่อด้วย Logic การอัปโหลด S3 หรือส่ง Email
  // await uploadToS3(job.data.orderId, pdfBuffer);
  
  await browser.close();
  console.log(`Finished: ${job.data.orderId}`);
});
```
---
## ⚠️ ข้อควรระวังและการจัดการปัญหา (Gotchas & Pitfalls)
การรัน Headless Browser ในระดับ Production มีจุดที่ต้องระวังอย่างมาก ดังนี้:

1. ปัญหา Zombie Processes (Memory Leak)
ปัญหา: Puppeteer อาจปิด Browser ไม่สนิทในบางกรณี ทำให้เกิดโปรเซสค้างอยู่ใน RAM จนเต็ม (Zombie Process)

วิธีแก้: ต้องใช้คำสั่ง await browser.close() ในบล็อก finally เสมอ และควรใช้เครื่องมืออย่าง dumb-init ใน Docker เพื่อช่วยจัดการ Lifecycle ของโปรเซสลูก

2. ปัญหาฟอนต์ภาษาไทย (Font Rendering)
ปัญหา: เมื่อรันบน Linux Server หรือ Docker มักจะเจอปัญหาภาษาไทยกลายเป็นสี่เหลี่ยม (ToFu) เพราะไม่มีฟอนต์ติดตั้งอยู่ในระบบ

วิธีแก้: ต้องทำการติดตั้งฟอนต์ (เช่น fonts-thai-tlwg) ลงใน Docker Image หรือใช้การฝังฟอนต์ผ่าน @font-face แบบ Base64 ลงใน HTML โดยตรง

3. การจัดการ Resource ใน Docker (Sandbox Issue)
ปัญหา: Puppeteer ต้องการ Sandbox ในการทำงาน ซึ่งมักจะรันไม่ได้ใน Docker ทั่วไปเนื่องจากติด Permission

วิธีแก้: ต้องตั้งค่า Flag { args: ['--no-sandbox', '--disable-setuid-sandbox'] } และระบุ Memory Limit ของ Container ให้เพียงพอ (อย่างน้อย 1-2GB ต่อ 1 Worker)

4. ปัญหาคิวค้าง (Job Timeout/Stuck)
ปัญหา: งานสร้าง PDF บางชิ้นอาจค้าง (เช่น ดึงรูปจากภายนอกไม่ได้) ทำให้ Worker ตัวนั้นทำงานต่อไม่ได้

วิธีแก้: ต้องตั้งค่า Timeout ให้กับ Job และ Browser รวมถึงตั้งค่า Retries (การลองใหม่) ใน BullMQ เพื่อให้ระบบดึงงานกลับมาทำใหม่หากเกิดข้อผิดพลาดชั่วคราว
