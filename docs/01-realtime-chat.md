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
```
---

## 🏗️ สถาปัตยกรรมระดับลึก (In-depth Architecture)

การสร้างระบบที่รองรับผู้ใช้งานจำนวนมหาศาลต้องเผชิญกับความท้าทายหลัก 3 ด้าน:

### 1. การจัดการการเชื่อมต่อข้ามโหนด (Node Intercommunication)
เมื่อระบบมี Server หลาย Node (Horizontal Scaling) ผู้ใช้ที่อยู่คนละ Server จะคุยกันไม่ได้หากไม่มีตัวกลาง
* **Solution:** ใช้ **Redis Pub/Sub** หรือ **Redis Streams** เป็น Message Broker กลาง
* **Why Streams?** เพราะรองรับการจองคิวข้อความ (Persistence) ป้องกันข้อความหายระหว่างที่ Network กระตุก (Transient Failure)



### 2. โครงสร้างพื้นฐาน Network
* **Load Balancer:** ต้องรองรับ Layer 7 และทำ **Sticky Sessions (Session Affinity)** เพื่อให้การทำ WebSocket Handshake สำเร็จที่โหนดเดิมเสมอ
* **Protocol:** ควรใช้ WSS (WebSocket Secure) เพื่อความปลอดภัยและหลีกเลี่ยงการโดน Block จาก Proxy บางตัว

### 3. การขยายฐานข้อมูล (Scalable Storage)
* **Write-Heavy Load:** ข้อมูลแชทเน้นการเขียน (Write) มหาศาล
* **Storage Choice:** **ScyllaDB** หรือ **Cassandra** เหมาะที่สุดเพราะทำ Sharding ตาม `chat_room_id` ได้ดีเยี่ยม ทำให้การดึง History ของห้องนั้นๆ ทำได้รวดเร็ว (Low Latency)

---

## 🛠️ แนะนำ Open Source สำหรับศึกษาต่อ
หากไม่อยากเริ่มจากศูนย์ (Build from scratch) แนะนำให้ศึกษาโปรเจกต์เหล่านี้:
1. **[Centrifugo](https://github.com/centrifugal/centrifugo):** Real-time messaging server ที่ Scalable มาก (เขียนด้วย Go)
2. **[Socket.io](https://socket.io/):** มาตรฐานสำหรับ Node.js (ต้องใช้คู่กับ Redis Adapter)
3. **[NATS.io](https://nats.io/):** Cloud-native messaging system ที่เร็วกว่า Redis Pub/Sub ในบาง Scenario

---

## 🧪 Lab: Distributed Chat with Redis Streams
Lab นี้จะจำลองการรัน Chat Server 2 ตัวที่คุยกันผ่าน Redis

### 1. ไฟล์ `docker-compose.yaml`
```yaml
version: '3.8'
services:
  redis:
    image: redis:alpine
    ports: ["6379:6379"]

  chat-node-1:
    image: node:18-alpine
    command: sh -c "npm install redis && node server.js"
    environment:
      - PORT=3001
      - NODE_ID=SERVER_A
    volumes: ["./:/app"]
    working_dir: /app
    ports: ["3001:3001"]

  chat-node-2:
    image: node:18-alpine
    command: sh -c "npm install redis && node server.js"
    environment:
      - PORT=3002
      - NODE_ID=SERVER_B
    volumes: ["./:/app"]
    working_dir: /app
    ports: ["3002:3002"]
```
2. ไฟล์ server.js (Logic ของ Consumer)
```JavaScript
const redis = require('redis');
const client = redis.createClient({ url: 'redis://redis:6379' });

async function start() {
  await client.connect();
  console.log(`[${process.env.NODE_ID}] Connected to Redis`);

  let lastId = '$'; // อ่านเฉพาะข้อความใหม่ที่เข้ามาหลังจาก Start

  while (true) {
    const results = await client.xRead(
      redis.commandOptions({ isolated: true }),
      { key: 'chat_stream', id: lastId },
      { COUNT: 1, BLOCK: 5000 }
    );

    if (results) {
      results.forEach(result => {
        result.messages.forEach(msg => {
          console.log(`[${process.env.NODE_ID}] New Msg: ${msg.data.text}`);
          lastId = msg.id; // Update ID ล่าสุดที่อ่านแล้ว
        });
      });
    }
  }
}
start();
```
## ⚠️ ข้อควรระวังและการจัดการปัญหา (Gotchas & Pitfalls)

การพัฒนาระบบ Realtime ที่รองรับผู้ใช้งานจำนวนมหาศาลมักเต็มไปด้วยความท้าทายที่คาดไม่ถึง นี่คือประเด็นสำคัญที่คุณต้องออกแบบระบบเพื่อรองรับ:

### 1. การจัดการการเชื่อมต่อใหม่ (Connection Storm / Thundering Herd)
* **ปัญหา:** หาก Server ล่มพร้อมกัน และเมื่อกลับมาออนไลน์ ผู้ใช้งานทุกคนจะพยายาม Reconnect ทันที ทำให้เกิด **Load Spike** จนระบบล่มซ้ำ (Cascading Failure)
* **วิธีแก้:** ใช้เทคนิค **Exponential Backoff with Jitter** โดยให้ฝั่ง Client สุ่มระยะเวลาในการหน่วงการ Retry เพื่อกระจายโหลดไม่ให้เข้าสู่เซิร์ฟเวอร์พร้อมกันทั้งหมด

### 2. การเรียงลำดับข้อความ (Message Ordering)
* **ปัญหา:** ในระบบ Distributed ข้อความอาจเดินทางถึงปลายทางไม่ตรงลำดับเนื่องจาก Network Latency (เช่น ข้อความที่ 2 มาถึงก่อนข้อความที่ 1)
* **วิธีแก้:** ฝั่ง Server ควรแนบ **Global Timestamp** หรือใช้ **Sequence Number** จากตัวกลางเดียวกัน (เช่น Redis Atomic Counter หรือ Snowflake ID) เพื่อให้ Client ใช้เรียงลำดับบนหน้าจอได้ถูกต้อง

### 3. การจัดการ Memory รั่วไหล (Memory Leaks ใน WebSocket)
* **ปัญหา:** การเปิด Connection ทิ้งไว้โดยไม่มีการ Cleanup เมื่อ User ปิดแอปฯ หรือเน็ตหลุด จะทำให้ Connection ค้างใน RAM จนหน่วยความจำเต็ม
* **วิธีแก้:** ต้องทำระบบ **Heartbeat (Ping/Pong)** เพื่อตรวจสอบสถานะการเชื่อมต่อ และตั้งค่า **Timeout** เพื่อตัดการเชื่อมต่อทันทีหากไม่ได้รับสัญญาณตอบกลับ

### 4. การจัดการสถานะออนไลน์ (Presence System)
* **ปัญหา:** การ Query Database โดยตรงเพื่อเช็คสถานะออนไลน์นั้นช้าเกินไปและสิ้นเปลืองทรัพยากร
* **วิธีแก้:** ใช้ **Redis Key (Set/Hash) แบบมี TTL (Time-to-Live)** เพื่อเก็บสถานะชั่วคราว และใช้ **Redis Pub/Sub** แจ้งเตือนเพื่อนร่วมห้องแชทแบบ Realtime เมื่อมีคน Login/Logout
