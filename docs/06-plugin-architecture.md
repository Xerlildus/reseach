## 🔌 6. Third-party Plugin Architecture
โครงสร้างการเรียกใช้ Plugin ภายนอกอย่างปลอดภัย (Simplified Pattern)
* **Sandboxing:** รัน Code ของบุคคลที่สามในสภาพแวดล้อมจำกัด เช่น **WebAssembly (WASM)** เพื่อความปลอดภัย
* **Hooks System:** สร้างจุดเชื่อมต่อ (Events) เช่น `before_payment_process` หรือ `after_user_signup`
* **Example:** **E-commerce Platform** - เปิดให้ Developer ภายนอกสร้าง Plugin "คำนวณค่าขนส่งตามระยะทาง" หรือ "ระบบสะสมแต้ม" เพิ่มเองได้โดยไม่ต้องแก้ Core Code
```rust
// ตัวอย่างแนวคิดใน Rust เพื่อนำไปทำ Wasm Module
pub fn calculate_tax(amount: f64) -> f64 {
    amount * 0.07
}
```
