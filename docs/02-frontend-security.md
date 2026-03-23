# 🔐 2. Frontend Security (SDK & API Key)

การจัดการความลับ (Secrets) ในฝั่ง Frontend เป็นเรื่องละเอียดอ่อน หลักการพื้นฐานคือ **"อย่าเชื่อใจ Client"** (Never trust the client) และนี่คือแนวทางปฏิบัติมาตรฐาน:

## 🛡️ กลยุทธ์การป้องกัน (Security Strategies)

### 1. การใช้ BFF (Backend-for-Frontend)
เพื่อหลีกเลี่ยงการเปิดเผย **Secret Key** ไว้ที่ Browser หรือ Mobile App เราควรสร้าง Layer กลางขึ้นมา
* **Concept:** Frontend คุยกับ Backend ของเราเอง (BFF) -> Backend ของเราถือ Secret Key และคุยกับ Third-party Service (เช่น Stripe, AWS, Firebase) อีกที
* **Benefit:** Secret Key จะถูกเก็บไว้ใน Environment Variable ของ Server เท่านั้น ไม่หลุดไปถึง User

### 2. Scoped Tokens & STS (Security Token Service)
ในกรณีที่จำเป็นต้องให้ Frontend คุยกับ Service โดยตรง (เช่น การ Upload ไฟล์ขึ้น S3) ให้ใช้การออก **Temporary Token**
* **สิทธิ์จำกัด:** Token นั้นต้องทำงานได้เฉพาะอย่าง (เช่น Write Only)
* **อายุสั้น:** กำหนดวันหมดอายุให้เร็วที่สุด (เช่น 5-15 นาที) เพื่อลดความเสี่ยงหาก Token ถูกขโมย

---

## 💻 ตัวอย่างการติดตั้ง (Implementation Lab)

### Scenario: ระบบชำระเงิน (Payment Gateway)
เราจะแยกการทำงานเป็น 2 ส่วน เพื่อให้แน่ใจว่า **Private Key** จะไม่หลุดไปที่ Frontend

#### 🟦 ฝั่ง Client (Frontend)
ทำหน้าที่รับข้อมูลบัตรและแลกเป็น `Token` (ซึ่งไม่มีมูลค่าในตัวมันเองหากไม่มี Private Key)

```javascript
// ตัวอย่างการใช้ Stripe Elements ในฝั่ง Frontend
const stripe = Stripe('pk_test_public_key_เท่านั้น'); // ใช้ Public Key

async function handlePayment() {
  const { token, error } = await stripe.createToken(cardElement);
  if (token) {
    // ส่ง Token ไปที่ BFF (Backend ของเรา) ไม่ใช่ส่งไปที่ Stripe Direct
    const response = await fetch('/api/charge', {
      method: 'POST',
      body: JSON.stringify({ token: token.id, amount: 1000 })
    });
  }
}
```
🟩 ฝั่ง Server (BFF - Node.js)
ทำหน้าที่ถือ Secret Key และสั่งตัดเงินจริง
```JavaScript
// Backend (Express.js) - นี่คือที่ที่ความลับถูกเก็บไว้
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

app.post('/api/charge', async (req, res) => {
  const { amount, token } = req.body;

  try {
    // การสั่ง Charge จริงจะเกิดขึ้นที่ Server-to-Server เท่านั้น
    const charge = await stripe.charges.create({
      amount: amount,
      currency: 'thb',
      source: token,
      description: 'Lab Payment Example',
    });

    res.status(200).json({ success: true, chargeId: charge.id });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});
```
---

## 🧪 Lab: Secure File Upload (AWS S3 Presigned URL)
หากต้องการให้ User อัปโหลดไฟล์ขนาดใหญ่โดยไม่ผ่าน Server ของเรา (เพื่อประหยัด Bandwidth) แต่ยังต้องการความปลอดภัย:

Frontend: ร้องขอ URL สำหรับอัปโหลดจาก Backend

Backend: ใช้ AWS SDK สร้าง Presigned URL (URL ชั่วคราวที่มีสิทธิ์เขียนไฟล์ได้แค่ 1 ไฟล์) แล้วส่งคืนให้ Frontend

Frontend: ใช้ URL นั้นทำการ PUT ไฟล์ขึ้น S3 โดยตรง

---

## ⚠️ ข้อควรระวังและการจัดการปัญหา (Gotchas & Pitfalls)
การจัดการ Security ในฝั่ง Frontend มักจะมีช่องโหว่ที่คาดไม่ถึง นี่คือประเด็นสำคัญที่คุณต้องตรวจสอบ:

1. การหลุดของ API Key ผ่าน Version Control
ปัญหา: เผลอ Push ไฟล์ .env หรือ Hardcoded Key ลงใน Public Repo (GitHub/GitLab) ทำให้ Bot สามารถสแกนและนำ Key ไปใช้งานจนโควต้าเต็มหรือสูญเสียเงิน

วิธีแก้: ใช้ไฟล์ .gitignore เพื่อยกเว้นไฟล์ .env เสมอ และหาก Key หลุดไปแล้ว ให้ทำการ Rotate Key (ยกเลิกอันเก่าและออกอันใหม่) ทันที

2. ช่องโหว่จากการตั้งค่า CORS (Cross-Origin Resource Sharing)
ปัญหา: การตั้งค่า Access-Control-Allow-Origin: * (อนุญาตทุกโดเมน) ทำให้เว็บไซต์ปลอมสามารถเรียกใช้ API ของคุณผ่าน Browser ของ User ได้ (CSRF Risk)

วิธีแก้: กำหนด Whitelist เฉพาะโดเมนที่ใช้งานจริงเท่านั้น (เช่น https://myapp.com) ในการตั้งค่า Middleware ของ Backend

3. การปลอมแปลงข้อมูลจากฝั่ง Client (Client-side Data Tampering)
ปัญหา: User แก้ไขจำนวนเงิน (Amount) หรือราคาสินค้าผ่าน Inspect Element ก่อนส่งมายัง API

วิธีแก้: ห้ามเชื่อข้อมูลราคาจาก Frontend ให้ส่งเฉพาะ ProductID มายัง Backend และให้ Backend ดึงราคาสินค้าจาก Database มาคำนวณเองเท่านั้น

4. การจัดการ Token หมดอายุ (Token Expiration Handling)
ปัญหา: ระบบ Scoped Token มีอายุสั้น ทำให้ User เจอ Error เมื่อ Token หมดอายุระหว่างการใช้งาน

วิธีแก้: ใช้ระบบ Silent Refresh โดยให้ Frontend ขอ Token ใหม่ใน Background ก่อนที่อันเดิมจะหมดอายุ หรือจัดการ Error Code 401 เพื่อ Redirect ไปยังหน้าขอสิทธิ์ใหม่โดยไม่ขัดจังหวะการใช้งาน (Seamless Experience)
