## 📄 4. PDF Invoice Generator
สร้างใบเสร็จรับเงินที่รองรับภาษาไทยด้วย Headless Browser
* **Headless Chrome:** ใช้ **Puppeteer** หรือ **Playwright** ในการ Render HTML/CSS ให้เป็น PDF เพื่อความยืดหยุ่นในการจัด Layout
* **Worker Queue:** ใช้ **BullMQ** หรือ **RabbitMQ** จัดคิวการสร้าง PDF เพื่อไม่ให้ Main API ทำงานหนักเกินไป
* **Example:** **ระบบ E-tax Invoice** - เมื่อลูกค้ากดซื้อสินค้า ระบบจะโยนงานเข้า Queue เพื่อให้ Worker สร้าง PDF ใบกำกับภาษีตาม Template และส่งอีเมลให้ลูกค้าอัตโนมัติ
```javascript
const browser = await puppeteer.launch();
const page = await browser.newPage();
await page.setContent('<h1>Invoice #123</h1><p>Total: 500 THB</p>');
await page.pdf({ path: 'invoice.pdf', format: 'A4' });
await browser.close();
```
