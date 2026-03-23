# 🤖 5. Claude Code & MCP (AI-Driven Development)

การใช้ AI เขียนโค้ดให้มีประสิทธิภาพสูงสุด ไม่ใช่การสั่งกว้างๆ แต่คือการให้ AI "เห็น" (Context) ในสิ่งที่จำเป็นต้องรู้ผ่านโปรโตคอลมาตรฐาน

## 🏗️ สถาปัตยกรรมระดับลึก (In-depth Architecture)

### 1. Model Context Protocol (MCP)
MCP คือมาตรฐานเปิดที่ช่วยให้ AI (เช่น Claude) สามารถเชื่อมต่อกับข้อมูลภายนอกได้โดยตรง
* **Direct Access:** แทนที่จะก๊อปปี้โค้ดไปวาง AI จะสามารถอ่านไฟล์ (File System), ค้นหาฐานข้อมูล (DB), หรือดึงข้อมูลจาก GitHub ได้เองผ่าน Local Server
* **Efficiency:** ลดข้อจำกัดเรื่อง Context Window เพราะ AI จะเลือกดึงเฉพาะข้อมูลที่เกี่ยวข้อง (Relevant Context) มาใช้งานเท่านั้น

### 2. Contextual Awareness Strategy
การให้ Context ที่ดีช่วยลดอาการ "หลอน" (Hallucination) ของ AI ได้มหาศาล
* **Selective Context:** ใช้คำสั่งอย่าง `/add` หรือระบุ Folder เฉพาะส่วน เพื่อให้ AI โฟกัสเฉพาะ Logic ที่เกี่ยวข้อง
* **Iterative Refactoring:** การเปลี่ยนโค้ดชุดใหญ่ (เช่น PHP ไปเป็น Node.js) ควรทำแบบทีละโมดูล (Module-by-module) โดยให้ AI เขียน Test คลุมไว้ก่อนเสมอ

---

## 🧪 Lab: Refactoring Legacy Code with Claude CLI

Lab นี้จะจำลองการใช้ Claude Code (CLI) เพื่อทำความสะอาดโค้ดเก่าและเพิ่ม Unit Test แบบอัตโนมัติ

### 🟩 Step 1: การเตรียม Context
สั่งให้ Claude อ่านโครงสร้างโปรเจกต์และกฎการเขียนโค้ด (Linting/Rules) ก่อนเริ่มงาน

```bash
# สั่งให้ Claude วิเคราะห์โครงสร้างโปรเจกต์ทั้งหมด
claude-code analyze ./src --summary

# เพิ่มไฟล์ที่เกี่ยวข้องเข้าสู่ Context (เช่น Controller และ Database Schema)
/add ./src/controllers/userController.js ./src/models/userModel.js
```
🟧 Step 2: การสั่ง Refactor และสร้าง Test
ส่งคำสั่งที่ชัดเจนเพื่อให้ AI ทำงานเป็นขั้นตอน
```bash
# สั่งให้แปลง Logic จาก callback เป็น async/await และเขียน Test
claude-code rewrite "Refactor userController to use async/await and add Jest unit tests in Thai language"
```
🟦 Step 3: Review และ Apply
AI จะเสนอความเปลี่ยนแปลง (Diff) ให้เราตรวจสอบก่อนกดกดยืนยันการแก้ไขไฟล์จริง
---
## ⚠️ ข้อควรระวังและการจัดการปัญหา (Gotchas & Pitfalls)
การให้ AI เข้าถึงระบบไฟล์ (File Access) มีความเสี่ยงและข้อจำกัดที่ต้องจัดการ:

1. ความปลอดภัยของข้อมูล (Data Privacy & Security)
ปัญหา: AI อาจอ่านไฟล์ที่มีความลับ (เช่น .env หรือ id_rsa) และส่งข้อมูลกลับไปยัง Server ของผู้ให้บริการเพื่อประมวลผล

วิธีแก้: ต้องตั้งค่าไฟล์ .claudeignore หรือระบุสิทธิ์ (Permission) ให้ AI เข้าถึงได้เฉพาะ Folder ที่เป็น Source Code เท่านั้น ห้ามให้สิทธิ์ Root เด็ดขาด

2. การแก้ไขโค้ดผิดพลาดแบบ Cascading (Unintended Changes)
ปัญหา: AI อาจแก้ไขโค้ดในจุดหนึ่ง แต่อีกจุดที่เรียกใช้งานกลับไม่ได้ถูกแก้ไขตาม ทำให้ระบบรันไม่ได้ (Build Break)

วิธีแก้: ต้องมี Automated Test (CI/CD) คอยตรวจสอบเสมอ และควรสั่งให้ AI ทำงานทีละส่วน (Small Incremental Changes) แทนการสั่งแก้ทั้งโปรเจกต์ในครั้งเดียว

3. ปัญหา Context Overload (Hallucination)
ปัญหา: หากใส่ไฟล์เข้าไปใน Context มากเกินไป AI จะเริ่มสับสนและให้คำแนะนำที่ขัดแย้งกับ Logic เดิมของระบบ

วิธีแก้: ใช้เทคนิค "Less is More" โดยการลบ Context ที่ไม่เกี่ยวข้องออกระหว่างการสนทนา และใช้คำสั่งสรุป (Summarize) เป็นระยะเพื่อให้ AI กลับมาโฟกัสที่เป้าหมายหลัก

4. การพึ่งพา AI มากเกินไป (Loss of Code Ownership)
ปัญหา: Developer กดยอมรับ (Accept) โค้ดที่ AI เขียนโดยไม่ได้อ่านทำความเข้าใจ ทำให้เมื่อเกิด Bug ในอนาคตจะแก้ไขได้ยากมาก

วิธีแก้: ต้องถือคติ "AI Proposes, Human Disposes" (AI เสนอ มนุษย์ตัดสิน) ทุกบรรทัดที่ AI เขียน มนุษย์ต้องผ่านการ Code Review และเข้าใจ Logic นั้นจริงๆ ก่อนนำขึ้น Production
