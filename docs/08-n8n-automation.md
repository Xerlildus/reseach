## ⚙️ 8. การทำ Workflow Automation ด้วย n8n
n8n เป็นเครื่องมือที่ทรงพลังสำหรับการทำ Workflow Automation ในรูปแบบ Node-based ซึ่งรองรับการ Self-hosted ทำให้คุณสามารถควบคุมข้อมูลได้เอง 100%
* **Self-Hosted:** ติดตั้งผ่าน Docker บน Server ตัวเองเพื่อความปลอดภัยของข้อมูลและความประหยัด
* **Webhooks:** ใช้ n8n เป็นตัวรับ Data จากแอปหนึ่ง (เช่น Typeform) แล้วส่งไปประมวลผลต่อในแอปอื่น (เช่น Google Sheet + ChatGPT)
* **Example:** **HR Automation** - เมื่อมีคนกรอกใบสมัคร -> n8n ส่ง Resume ไปให้ AI สรุปคะแนน -> ถ้าคะแนนสูงส่งแจ้งเตือนเข้า Slack ของทีม HR
การสร้าง Workflow ใน n8n มักใช้ Node ต่อกัน:  
Webhook Node: รับข้อมูลจาก Form  
AI Agent Node: ส่งข้อมูลไปวิเคราะห์ที่ OpenAI/Claude  
Slack Node: ส่งผลลัพธ์เข้าช่องทางทีม

## สถาปัตยกรรม Workflow
การสร้าง Workflow ใน n8n มักจะเชื่อมต่อ Node ต่อกันตามลำดับดังนี้:
* **Webhook Node:** รับข้อมูลจาก Form หรือจากระบบภายนอก (Trigger)
* **AI Agent Node:** ส่งข้อมูลไปวิเคราะห์ที่ LLM (OpenAI/Claude) เพื่อสรุปเนื้อหาหรือวิเคราะห์ข้อมูล
* **Slack/Email Node:** ส่งผลลัพธ์ที่ได้เข้าสู่ช่องทางติดต่อทีมงาน  
## คำแนะนำเชิงเทคนิค (Best Practices)  
* **Self-hosted via Docker:** ควรติดตั้งผ่าน Docker Compose บน Server ของคุณเองเพื่อความปลอดภัย
* **Scaling:** สำหรับระดับองค์กร ให้ใช้ n8n Queued Mode ซึ่งใช้ Redis ในการจัดการคิวงาน ช่วยให้ Worker หลายตัวช่วยกันทำงานได้โดยไม่ทำให้ระบบค้าง
* **Idempotency:** ออกแบบ Workflow ให้สามารถทำงานซ้ำได้โดยไม่ก่อให้เกิด Duplicate Data (เช่น การเช็ค Record ใน DB ก่อนสร้างใหม่)

