## 🤖 5. Claude Code Tricks
ใช้ MCP (Model Context Protocol) เพื่อให้ AI เข้าถึงระบบไฟล์ได้
* **Contextual Awareness:** ใช้คำสั่ง `/add` เพื่อเลือกเฉพาะไฟล์ที่เกี่ยวข้องกับ Logic นั้นๆ ลดการหลอน (Hallucination) ของ AI
* **Iterative Refactoring:** สั่งให้ Claude วิเคราะห์หาจุดคอขวด (Bottleneck) และเสนอแนวทางแก้เป็นขั้นตอน (Step-by-step)
* **Example:** **Legacy Code Refactor** - สั่งให้ Claude อ่านโค้ด PHP เก่าแล้วแปลงเป็น NestJS (TypeScript) พร้อมเขียน Unit Test ให้ทันที
```javascript
# ตัวอย่างการรัน Claude Code CLI เพื่อทำ Code Review ทั้งโปรเจกต์
claude-code analyze ./src --rules "Ensure all functions have JSDoc and Thai language support"
```
