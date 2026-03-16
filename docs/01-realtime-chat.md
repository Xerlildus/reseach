# 💬 1. ระบบแชท Realtime และ Scalable



## สถาปัตยกรรมระดับลึก (In-depth Architecture)
การสร้างระบบแชทที่รองรับผู้ใช้งานหลักล้าน (Million concurrent users) ต้องเผชิญกับความท้าทายเรื่อง "Connection Storm" และการรักษาลำดับของข้อความ (Message Consistency)

### 1. การจัดการการเชื่อมต่อข้ามโหนด
เมื่อระบบมี Server หลาย Node ผู้ใช้ A (เชื่อมต่อโหนด 1) จะส่งข้อความหาผู้ใช้ B (เชื่อมต่อโหนด 2) ได้อย่างไร?
* **แนวทางแก้ไข:** ใช้ **Redis Pub/Sub** เป็นตัว Broadcaster กระจายข้อความระหว่างโหนด แต่สำหรับระบบที่ต้องการความแม่นยำสูง ควรเปลี่ยนมาใช้ **Redis Streams** เพื่อป้องกันข้อความสูญหายหาก Server มีการเชื่อมต่อหลุด (Transient failure)

### 2. โครงสร้างพื้นฐานของ Network
* **Load Balancer & Sticky Sessions:** หัวใจสำคัญคือการทำ Sticky Sessions (Session Affinity) เพื่อให้ WebSocket Handshake สำเร็จ โดยควรเลือกใช้ Nginx หรือ Load Balancer ที่รองรับ Protocol WebSocket โดยตรง

### 3. การขยายฐานข้อมูล (Scalable Storage)
เมื่อข้อมูลแชทมีปริมาณมหาศาล:
* **Database Sharding:** ใช้ ScyllaDB หรือ Cassandra โดยทำ Sharding ข้อมูลตาม `chat_room_id` เพื่อกระจายโหลดไม่ให้กระจุกตัวอยู่ที่โหนดใดโหนดหนึ่ง

---
## ตัวอย่างการติดตั้งระบบ (Implementation)
```javascript
// ตัวอย่าง Publisher สำหรับส่งข้อความเข้าสู่ Redis Streams
const redis = require('redis');
const client = redis.createClient();

async function sendMessage(roomId, senderId, text) {
  await client.xAdd('chat_stream', '*', {
    roomId, senderId, text, timestamp: Date.now().toString()
  });
}
