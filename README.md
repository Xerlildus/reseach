# 🚀 Engineering & Automation Research Hub 2026
> A curated collection of modern architecture, security practices, and AI-driven automation.

---

## 📑 Table of Contents
* [💬 1. Realtime & Scalable Chat](#1-realtime--scalable-chat)
* [🔐 2. Frontend Security (SDK & API Key)](#2-frontend-security-sdk--api-key)
* [🖼️ 3. Watermarking (Image & File)](#3-watermarking-image--file)
* [📄 4. PDF Invoice Generator](#4-pdf-invoice-generator)
* [🤖 5. Claude Code Tricks](#5-claude-code-tricks)
* [🔌 6. Third-party Plugin Architecture](#6-third-party-plugin-architecture)
* [🔍 7. Elasticsearch Mastery](#7-elasticsearch-mastery)
* [⚙️ 8. n8n Automation Workflows](#8-n8n-automation-workflows)
* [🎥 9. Auto-generate Video Content](#9-auto-generate-video-content)

---

## 💬 1. Realtime & Scalable Chat
การออกแบบระบบ Chat ที่รองรับผู้ใช้หลักล้านด้วย **WebTransport** และ **Pub/Sub Mechanism**

### 🛠️ Key Architecture
* **WebTransport:** มาแทนที่ WebSocket เพื่อลด Latency ในระดับ Microseconds
* **Redis Pub/Sub:** ใช้กระจาย Message ข้าม Server Instances (Horizontal Scaling)
* **Persistence:** ใช้ **ScyllaDB** หรือ **Cassandra** สำหรับเก็บ Message ประวัติย้อนหลัง

### 💻 Code Example (Node.js + Redis Adapter)
```javascript
const { Server } = require("socket.io");
const { createAdapter } = require("@socket.io/redis-adapter");
const { createClient } = require("redis");

const pubClient = createClient({ url: "redis://localhost:6379" });
const subClient = pubClient.duplicate();

const io = new Server(3000);
io.adapter(createAdapter(pubClient, subClient));

io.on("connection", (socket) => {
  socket.on("chat_message", (msg) => io.emit("broadcast", msg));
});
```
🔐 2. Frontend Security (SDK & API Key)
การป้องกัน API Key รั่วไหลบน Client-side ด้วย BFF Pattern

🛡️ Best Practices
BFF (Backend-for-Frontend): Frontend ไม่เรียก API ตรงๆ แต่เรียกผ่าน Proxy ของตัวเอง

App Check: ใช้ตัวตรวจสอบ Attestation เพื่อยืนยันว่า Request มาจากแอปจริงเท่านั้น

Domain Whitelisting: จำกัดการใช้งาน Key เฉพาะ Origin URL ที่ระบุ

💻 Implementation (Next.js Proxy)
```JavaScript
// pages/api/proxy.js
export default async function handler(req, res) {
  const response = await fetch('[https://api.external.com/data](https://api.external.com/data)', {
    headers: { 'Authorization': `Bearer ${process.env.PRIVATE_KEY}` }
  });
  res.status(200).json(await response.json());
}
```
🖼️ 3. Watermarking (Image & File)
ระบบป้องกันการคัดลอกและตรวจสอบที่มาของข้อมูล (Digital Forensics)

🧩 Methods
Visible: ใช้ Sharp (Node.js) ซ้อน Layer ลายน้ำแบบโปร่งแสง

Invisible: ฝัง Metadata ลงในพิกเซลภาพ (LSB) หรือปรับ Kerning ใน PDF

Dynamic: ฝัง User ID ลงในพื้นหลังเมื่อมีการ Preview ไฟล์

📄 4. PDF Invoice Generator
การสร้างเอกสารทางการที่รองรับภาษาไทย 100% และ Layout ที่แม่นยำ

🚀 Recommended Stack
Engine: Playwright หรือ Puppeteer (HTML-to-PDF)

Library: react-pdf สำหรับ Client-side หรือ jsPDF สำหรับงานง่ายๆ

Font: ต้องใช้ Google Fonts (เช่น Sarabun) และรองรับ Subset ภาษาไทย

🤖 5. Claude Code Tricks
เทคนิคการใช้ AI Agent จัดการ Codebase ขนาดใหญ่

💡 Effective Hacks
MCP (Model Context Protocol): เชื่อมต่อ Claude กับฐานข้อมูลในเครื่องหรือ GitHub โดยตรง

Task Decomposition: สั่งงานแยกส่วน เช่น Plan -> Implement -> Test

Custom Rules: ใช้ไฟล์ .claudecode เพื่อบังคับ Standard ของโปรเจกต์

🔌 6. Third-party Plugin Architecture
การออกแบบ Product ให้เป็น Platform ที่นักพัฒนาภายนอกมาต่อเติมได้

🏗️ Design Steps
Sandboxing: รัน Plugin ใน Iframe หรือ Web Worker เพื่อความปลอดภัย

Standard API: สร้าง PluginSDK ให้ Plugin เรียกใช้ฟังก์ชันในระบบหลัก

Event Hooks: ใช้ Webhooks เพื่อส่งข้อมูลเหตุการณ์ไปยัง Plugin

🔍 7. Elasticsearch Mastery
การค้นหาข้อมูลมหาศาลด้วยความเร็วระดับ Milliseconds

⚡ Critical Skills
Vector Search: ใช้เก็บ Embedding เพื่อทำ Semantic Search (ค้นหาด้วยความหมาย)

Analyzers: ตั้งค่า icu_analyzer เพื่อตัดคำภาษาไทยให้แม่นยำ

Kibana: ใช้ทำ Visualizer และ Monitoring สุขภาพของ Cluster

⚙️ 8. n8n Automation Workflows
การทำ Workflow Automation แบบ Low-code ที่ทรงพลัง

🌊 Workflow Pattern
Trigger: รับ Webhook หรือตั้งเวลา (Cron)

Logic: ใช้ Function Node (JavaScript) เมื่อ Logic ซับซ้อนเกินไป

Error Handling: สร้าง Error Node เพื่อส่ง Alert เข้า Slack เมื่อระบบพัง

🎥 9. Auto-generate Video Content
การสร้างวิดีโอระดับ Production ผ่านโค้ดและ AI

🎬 Pipeline
Script: ใช้ Claude สร้าง Content JSON

Audio: ใช้ ElevenLabs API พากย์เสียงภาษาไทย

Render: ใช้ Remotion (React) ในการ Render ภาพและเสียงเป็น MP4