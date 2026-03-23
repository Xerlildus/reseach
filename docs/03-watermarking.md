# 🖼️ 3. Watermarking (Image & File Protection)

การป้องกันการนำรูปภาพหรือเอกสารไปใช้โดยไม่ได้รับอนุญาต ไม่ได้มีแค่การทำเครื่องหมายทับภาพ แต่รวมถึงการฝังข้อมูลที่ระบุตัวตนผู้เข้าชมเพื่อป้องกันการหลุดของข้อมูล (Data Leak)

## 🏗️ สถาปัตยกรรมระดับลึก (In-depth Architecture)

### 1. On-the-fly Processing (Dynamic Watermarking)
แทนที่จะทำลายน้ำทิ้งไว้ทุกลูกแบบ Static เราสามารถสร้างลายน้ำ "ตามชื่อผู้ใช้งาน" (User-specific) ได้ทันทีที่เรียกดู
* **Processing Engine:** ใช้ Library อย่าง **Sharp** (Node.js) หรือ **Pillow** (Python) รันบน Server หรือ Serverless Function (Lambda)
* **Image CDN:** ใช้บริการอย่าง **Cloudinary** หรือ **Imgix** ที่รองรับการเติม Overlay ผ่าน URL Parameters เช่น `image.jpg?overlay=user_id_123`

### 2. Invisible Watermarking (Steganography)
เทคนิคการฝังรหัสลับลงในพิกเซลของภาพหรือ Metadata ซึ่งมองไม่เห็นด้วยตาเปล่า
* **Metadata:** ฝังข้อมูลลงใน EXIF Data ของไฟล์
* **Frequency Domain:** การปรับเปลี่ยนค่าสีในระดับที่ลึกซึ้ง (Bit manipulation) ทำให้แม้จะถูกนำไป Crop หรือ Resize ลายน้ำก็ยังสามารถถูกถอดรหัสออกมาได้เพื่อยืนยันต้นทาง

---

## 🧪 Lab: Dynamic Watermark with Sharp

Lab นี้จะจำลองการรับ Request รูปภาพแล้วนำชื่อ User มาปั๊มเป็นลายน้ำก่อนส่งกลับไป (Buffer Stream)

```javascript
const sharp = require('sharp');
const express = require('express');
const app = express();

app.get('/view-image/:imageName', async (req, res) => {
  const userName = req.query.user || 'Guest'; // รับชื่อ User จาก Session/Query
  const imagePath = `./assets/${req.params.imageName}`;

  try {
    // 1. สร้างลายน้ำจาก Text (SVG)
    const svgWatermark = Buffer.from(`
      <svg width="400" height="200">
        <style>
          .title { fill: rgba(255, 255, 255, 0.3); font-size: 24px; font-weight: bold; }
        </style>
        <text x="50%" y="50%" text-anchor="middle" class="title">Viewed by: ${userName}</text>
      </svg>
    `);

    // 2. ใช้ Sharp ผสมภาพ (Composition)
    const processedImage = await sharp(imagePath)
      .composite([{ input: svgWatermark, gravity: 'center' }])
      .jpeg()
      .toBuffer();

    // 3. ส่งภาพกลับไปที่ Browser โดยไม่เก็บไฟล์ถาวร
    res.set('Content-Type', 'image/jpeg');
    res.send(processedImage);

  } catch (err) {
```
---
## ⚠️ ข้อควรระวังและการจัดการปัญหา (Gotchas & Pitfalls)
การทำ Watermarking มีผลกระทบต่อประสิทธิภาพและประสบการณ์ผู้ใช้ที่ต้องจัดการให้ดี:

1. ภาระของ CPU (High CPU Overhead)
ปัญหา: การประมวลผลภาพ (Image Processing) ทุกครั้งที่มีคนกดดู จะกิน CPU มหาศาล หากมีคนเข้าชมพร้อมกันจำนวนมาก Server อาจจะค้างได้

วิธีแก้: ต้องใช้ระบบ Caching (เช่น Redis หรือ CDN) เพื่อเก็บรูปที่ประมวลผลแล้วไว้ระยะหนึ่ง หรือใช้ Serverless (Lambda) ที่ขยายตัวได้ตามจำนวน Request เพื่อไม่ให้กระทบ Server หลัก

2. ความสมดุลระหว่างความปลอดภัยและความสวยงาม (UX vs Security)
ปัญหา: ลายน้ำที่เข้มเกินไปทำให้มองภาพไม่ชัด (User รำคาญ) แต่ลายน้ำที่จางหรือเล็กเกินไปก็ถูกลบออกได้ง่ายด้วย AI (AI Inpainting/Object Removal)

วิธีแก้: ใช้ Tiled Watermark (ลายน้ำแบบซ้ำๆ ทั่วทั้งภาพ) ที่มีความโปร่งแสง (Opacity) ประมาณ 10-15% ซึ่งยากต่อการลบออกด้วยโปรแกรมแต่งภาพทั่วไป

3. การป้องกันการ Bypass URL
ปัญหา: หากใช้ Image CDN (เช่น Cloudinary) User หัวหมออาจจะลบ Parameter ท้าย URL ออกเพื่อดูภาพต้นฉบับที่ไม่มีลายน้ำ

วิธีแก้: ต้องใช้เทคนิค Signed URLs โดยให้ Backend สร้าง Hash กำกับ URL ไว้ หาก User แก้ไข URL แม้แต่นิดเดียว URL นั้นจะใช้งานไม่ได้ทันที

4. ปัญหาเรื่อง Metadata Privacy
ปัญหา: การส่งรูปภาพต้นฉบับที่มี EXIF Data ครบถ้วน อาจหลุดข้อมูล Location (GPS) ของผู้ถ่าย ซึ่งเป็นความเสี่ยงด้านความเป็นส่วนตัว

วิธีแก้: ในขั้นตอนการทำ Watermark ควรตั้งค่าสั่ง .withMetadata(false) หรือลบข้อมูลที่ไม่จำเป็นออกก่อนส่งให้ Client ทุกครั้ง
    res.status(404).send('Image not found');
  }
});
