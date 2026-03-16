## 💬 1. Realtime & Scalable Chat
ออกแบบด้วยระบบ Pub/Sub เพื่อรองรับการขยายตัวแบบ Horizontal Scaling
* **Architecture:** ใช้ **WebSockets (WS)** สำหรับการเชื่อมต่อ และ **Redis Pub/Sub** เป็นตัวกลาง (Message Broker) เพื่อให้ Server แต่ละ Node คุยกันได้
* **State Management:** ไม่เก็บ Session ไว้ที่ Server ตัวใดตัวหนึ่ง (Stateless) แต่ใช้ **Shared State** บน Redis
* **Database:** ใช้ **ScyllaDB** หรือ **Cassandra** สำหรับเก็บ Chat History ปริมาณมหาศาลเนื่องจากรองรับ High Write Throughput
* **Example:** **แอปฯ ส่งอาหาร (Food Delivery)** - ไรเดอร์ส่งพิกัด GPS แบบ Real-time ผ่าน Pub/Sub และเก็บประวัติการคุยกับลูกค้าลง Database แยกกัน
```javascript
// Publisher
const redis = require('redis');
const pub = redis.createClient();
pub.publish('chat_channel', JSON.stringify({ user: 'A', msg: 'Hello!' }));

// Subscriber (บนทุก Server Node)
const sub = redis.createClient();
sub.subscribe('chat_channel');
sub.on('message', (channel, message) => {
  io.emit('chat_msg', JSON.parse(message)); // ส่งให้ Client ที่ต่อกับโหนดนี้
});
```
