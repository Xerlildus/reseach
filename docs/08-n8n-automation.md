## ⚙️ 8. n8n Automation Workflows
ตัวอย่าง Function Node (JavaScript) ใน n8n เพื่อกรองข้อมูล
* **Self-Hosted:** ติดตั้งผ่าน Docker บน Server ตัวเองเพื่อความปลอดภัยของข้อมูลและความประหยัด
* **Webhooks:** ใช้ n8n เป็นตัวรับ Data จากแอปหนึ่ง (เช่น Typeform) แล้วส่งไปประมวลผลต่อในแอปอื่น (เช่น Google Sheet + ChatGPT)
* **Example:** **HR Automation** - เมื่อมีคนกรอกใบสมัคร -> n8n ส่ง Resume ไปให้ AI สรุปคะแนน -> ถ้าคะแนนสูงส่งแจ้งเตือนเข้า Slack ของทีม HR
การสร้าง Workflow ใน n8n มักใช้ Node ต่อกัน:  
Webhook Node: รับข้อมูลจาก Form  
AI Agent Node: ส่งข้อมูลไปวิเคราะห์ที่ OpenAI/Claude  
Slack Node: ส่งผลลัพธ์เข้าช่องทางทีม
