# 🔌 6. Third-party Plugin Architecture (Platform Extensibility)

การเปิดให้ Developer ภายนอกเข้ามาเขียนส่วนขยาย (Plugins) ช่วยให้ระบบเติบโตอย่างก้าวกระโดด แต่ต้องแลกมาด้วยความเสี่ยงด้านความปลอดภัยและเสถียรภาพของระบบหลัก (Core System)

## 🏗️ สถาปัตยกรรมระดับลึก (In-depth Architecture)

### 1. Sandboxing with WebAssembly (WASM)
แทนที่จะรันโค้ดของคนอื่นบน Server เราโดยตรง (เสี่ยงต่อการโดน Remote Code Execution) เราจะรันผ่าน **WebAssembly** * **Isolation:** WASM ทำงานใน Sandbox ที่จำกัดการเข้าถึง File System, Network และ Memory ของเครื่อง Host
* **Performance:** ทำงานได้เร็วใกล้เคียงกับ Native Code และรองรับหลายภาษา (Rust, Go, C++, AssemblyScript)

### 2. Event-Driven Hooks System
Core System ต้องกำหนดจุดเชื่อมต่อ (Extension Points) ที่ชัดเจน เพื่อให้ Plugin เข้ามาแทรก Logic ได้
* **Action Hooks:** ทำงานเมื่อเกิดเหตุการณ์ (เช่น `on_order_created`)
* **Filter Hooks:** รับข้อมูลไปแก้ไขแล้วส่งคืนกลับมา (เช่น `filter_price_calculation`)



---

## 🧪 Lab: Secure Tax Calculator Plugin (WASM Concept)

Lab นี้จะจำลองการสร้าง Plugin คำนวณภาษีด้วย Rust และนำไปรันบน Node.js อย่างปลอดภัย

### 🦀 Step 1: เขียน Plugin ด้วย Rust (Compiled to WASM)
```rust
// plugin/src/lib.rs
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn calculate_tax(amount: f64, rate: f64) -> f64 {
    // Logic ของ Plugin ภายนอก
    amount * rate
}
```
🟩 Step 2: Core System (Node.js) เรียกใช้ Plugin
```Javascript
const fs = require('fs');
const { calculate_tax } = require('./pkg/tax_plugin.js'); // ไฟล์ที่ Compile มาแล้ว

function processOrder(amount) {
  const taxRate = 0.07;
  
  // เรียกใช้ Plugin ผ่าน Sandbox
  const totalTax = calculate_tax(amount, taxRate);
  
  console.log(`Amount: ${amount}, Tax: ${totalTax}, Total: ${amount + totalTax}`);
}

processOrder(1000);
```
---
## ⚠️ ข้อควรระวังและการจัดการปัญหา (Gotchas & Pitfalls)
การเปิดระบบให้บุคคลที่สามมีความเสี่ยงสูง นี่คือจุดที่คุณต้องคุมเข้ม:

1. ปัญหา Resource Exhaustion (Infinite Loops)
ปัญหา: Plugin ของบุคคลที่สามอาจมีบั๊กที่ทำให้เกิด Infinite Loop หรือกิน CPU 100% จนทำให้ Core System ค้างไปด้วย

วิธีแก้: ต้องมีการกำหนด Execution Timeout และ Memory Limits ให้กับ Sandbox (เช่น หาก Plugin รันนานเกิน 500ms ให้สั่ง Terminate ทันที)

2. ช่องโหว่ด้านความปลอดภัย (Data Exfiltration)
ปัญหา: Plugin พยายามแอบดึงข้อมูลลูกค้า (Sensitive Data) หรือส่งข้อมูลออกไปยัง Server ภายนอกโดยไม่ได้รับอนุญาต

วิธีแก้: ใช้หลักการ Least Privilege โดยให้ Plugin เข้าถึงเฉพาะข้อมูลที่จำเป็นผ่าน API ที่เรากำหนดเท่านั้น (ห้ามให้เข้าถึง Global Objects หรือ Environment Variables)

3. ปัญหา Version Compatibility (Breaking Changes)
ปัญหา: เมื่อเราอัปเดต Core System อาจทำให้ Plugin เดิมใช้งานไม่ได้ (Plugin Break) หรือในทางกลับกัน Plugin ที่เขียนไม่ดีอาจทำให้ Core ล่ม

วิธีแก้: ต้องทำ Semantic Versioning (SemVer) สำหรับ Plugin API และมีระบบ Compatibility Check เพื่อตรวจสอบเวอร์ชันของ Plugin ก่อนการเปิดใช้งาน (Activation)

4. การจัดการความล่าช้า (Latency Impact)
ปัญหา: หากระบบหลักต้องรอ (Blocking) ให้ Plugin ทำงานเสร็จก่อนถึงจะไปต่อได้ จะทำให้แอปฯ ช้าลงอย่างมาก

วิธีแก้: สำหรับงานที่ไม่ต้องการผลลัพธ์ทันที (เช่น การส่ง Log) ควรเปลี่ยนไปใช้ Asynchronous Hooks หรือ Webhook Queue เพื่อไม่ให้ Plugin มาดึงประสิทธิภาพของเส้นทางหลัก (Critical Path)
