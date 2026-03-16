## 🖼️ 3. Watermarking (Image & File)
การฝังลายน้ำลงบนรูปภาพก่อนส่งให้ Client
* **On-the-fly Processing:** ใช้ Library **Sharp** (Node.js) หรือ **Pillow** (Python) ประมวลผลภาพก่อนส่งให้ User หรือใช้ Image CDN อย่าง **Cloudinary** เพื่อทำลายน้ำผ่าน URL Parameters
* **Metadata Embedding:** ฝังข้อมูลเจ้าของไฟล์ลงใน Metadata หรือใช้เทคนิค **Digital Steganography** (ฝังลายน้ำแบบมองไม่เห็นด้วยตาเปล่า)
* **Example:** **Stock Photo Platform** - แสดงภาพตัวอย่างที่ติดลายน้ำแบบ Dynamic ตามชื่อ User ที่กำลังดู เพื่อป้องกันการแคปหน้าจอ
```javascript
const sharp = require('sharp');
await sharp('input.jpg')
  .composite([{ input: 'watermark.png', gravity: 'southeast' }])
  .toFile('output-watermarked.jpg');
```
