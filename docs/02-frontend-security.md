## 🔐 2. Frontend Security (SDK & API Key)
การใช้ BFF (Backend-for-Frontend) เพื่อซ่อนความลับ (Secrets)
* **Backend-for-Frontend (BFF):** หลีกเลี่ยงการแปะ Secret Key ไว้ที่ Frontend แต่ให้เรียกผ่าน API ของเราเองเพื่อซ่อน Key ไว้ที่ Environment Variable ของ Backend
* **Scoped Tokens:** ใช้ **STS (Security Token Service)** เพื่อออก Token ชั่วคราวที่มีอายุสั้นและจำกัดสิทธิ์ (เช่นใช้งานได้เฉพาะการอ่านภาพใน S3 Bucket เท่านั้น)
* **Example:** **ระบบจ่ายเงิน (Payment Gateway)** - Frontend รับเฉพาะ Public Key สำหรับสร้าง Token บัตรเครดิต แต่การสั่งตัดเงิน (Charge) จะทำผ่าน Backend ที่ถือ Private Key เท่านั้น
```javascript
// Backend (Express.js)
app.post('/api/charge', async (req, res) => {
  const { amount, token } = req.body;
  // เรียก Payment Gateway โดยดึง Secret จาก Environment Variable
  const charge = await stripe.charges.create({
    amount, currency: 'thb', source: token,
  }, { apiKey: process.env.STRIPE_SECRET_KEY });
  res.json(charge);
});
```
