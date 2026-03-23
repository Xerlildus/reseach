# ⚙️ 8. Workflow Automation ด้วย n8n (The Glue of Systems)

n8n เป็นเครื่องมือ Low-code Automation แบบ Node-based ที่ยืดหยุ่นสูง โดยเฉพาะการทำ Self-hosted ซึ่งช่วยให้เราควบคุม Data Privacy ได้ 100% เหมาะสำหรับการเชื่อมต่อ Microservices เข้าด้วยกัน

## 🏗️ สถาปัตยกรรมระดับลึก (In-depth Architecture)

### 1. Node-based Logic Flow
การสร้าง Workflow ใน n8n คือการนำ Node มาต่อกันเพื่อสร้าง Pipeline ข้อมูล:
* **Trigger Node:** จุดเริ่มต้น เช่น Webhook, Schedule (Cron), หรือเหตุการณ์ในแอปอื่น (เช่น New Typeform Submission)
* **Action Node:** การประมวลผล เช่น HTTP Request, การรัน JavaScript (Code Node), หรือการคุยกับฐานข้อมูล
* **AI Agent Node:** การส่งข้อมูลไปให้ LLM (Claude/OpenAI) เพื่อวิเคราะห์ วิพากษ์ หรือสรุปผลก่อนส่งต่อ

### 2. Enterprise Scaling (Queue Mode)
สำหรับการรัน Workflow จำนวนมหาศาล n8n รองรับการรันแบบ **Queue Mode**:
* **Redis:** ทำหน้าที่เป็น Message Broker คอยจ่ายงาน
* **Main Instance:** ทำหน้าที่จัดการ UI และรับ Webhook
* **Worker Instances:** ตัวประมวลผลงาน (Execution) ที่สามารถ Scale เพิ่มจำนวนได้ตามโหลดของงาน



---

## 🧪 Lab: AI-Powered Lead Scoring Workflow

Lab นี้จะจำลองการรับข้อมูลจากฟอร์มลูกค้า แล้วให้ AI วิเคราะห์ว่าควรส่งต่อให้เซลล์ท่านไหน

### 🟩 Step 1: Webhook & AI Analysis
1. **Webhook Node:** รับ JSON Data จากฟอร์ม (ชื่อบริษัท, งบประมาณ, ปัญหาที่พบ)
2. **AI Agent Node:** - **System Prompt:** "วิเคราะห์ข้อมูลลูกค้าและให้คะแนน 1-10 ตามศักยภาพ พร้อมระบุว่าควรส่งให้ทีม Enterprise หรือ SME"
   - **Model:** ใช้ Claude 3.5 Sonnet ผ่านรหัสลับใน Credential Manager

### 🟧 Step 2: Conditional Branching & Notification
3. **If Node:** ตรวจสอบคะแนนจาก AI หากคะแนน > 8 ให้ส่งไปที่ Enterprise Team
4. **Slack Node:** ส่งข้อความแจ้งเตือนเข้า Channel พร้อมสรุปจาก AI

---

## ⚠️ ข้อควรระวังและการจัดการปัญหา (Gotchas & Pitfalls)

การออกแบบระบบ Automation ที่มีความซับซ้อนต้องคำนึงถึงเสถียรภาพและความปลอดภัยเป็นสำคัญ:

### 1. ปัญหา Idempotency (การรันซ้ำแล้วเกิดข้อมูลซ้ำซ้อน)
* **ปัญหา:** หาก Workflow หยุดชะงักและมีการรันซ้ำ (Retry) อาจเกิดการสร้าง Order ซ้ำ หรือส่งอีเมลเดิมให้ลูกค้าหลายรอบ
* **วิธีแก้:** ต้องออกแบบให้มี **Check Point** เสมอ เช่น ใช้ Node "Database Query" เพื่อเช็คว่า `order_id` นี้ถูกประมวลผลไปหรือยังก่อนจะเริ่มงานถัดไป

### 2. การรั่วไหลของข้อมูลส่วนบุคคล (PII Leakage)
* **ปัญหา:** ข้อมูลชื่อ-นามสกุล หรือเบอร์โทรลูกค้า ถูกส่งไปยัง AI Model ภายนอกโดยตรง ซึ่งอาจขัดต่อนโยบาย PDPA/GDPR
* **วิธีแก้:** ใช้ **Code Node (JavaScript)** เพื่อทำการ **Data Redaction** (เซนเซอร์ข้อมูล) เช่น เปลี่ยนชื่อจริงเป็น "Customer A" ก่อนส่งให้ AI วิเคราะห์เฉพาะเนื้อหาทางธุรกิจ

### 3. การจัดการ Credential และ API Key
* **ปัญหา:** การใส่ API Key หรือ Password ไว้ใน Function Node โดยตรง (Hardcoded) มีความเสี่ยงสูงหากเผลอ Export Workflow ออกไปภายนอก
* **วิธีแก้:** ใช้ **n8n Credential Manager** เสมอ หรือดึงค่าผ่าน **Environment Variables** เพื่อให้การหมุนเวียน (Key Rotation) ทำได้ง่ายและปลอดภัยจากไฟล์ Config กลาง

### 4. Workflow Memory Exhaustion
* **ปัญหา:** การประมวลผลไฟล์ขนาดใหญ่ (เช่น รูปภาพหรือ PDF จำนวนมาก) ใน Workflow เดียวกันอาจทำให้ RAM ของ Node.js เต็มจน n8n Crash
* **วิธีแก้:** ใช้เทคนิค **Wait Node** เพื่อหน่วงเวลา หรือแยกงานใหญ่ๆ ออกเป็น **Sub-workflows** เพื่อให้แต่ละส่วนมีวงจรชีวิต (Lifecycle) ของ Memory แยกจากกัน
