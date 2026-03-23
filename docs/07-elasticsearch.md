# 🔍 7. Elasticsearch Mastery (Advanced Search)

เมื่อข้อมูลมีปริมาณมหาศาล การใช้คำสั่ง `LIKE %...%` ใน SQL จะทำให้ระบบช้าลงอย่างมาก Elasticsearch จึงเข้ามาช่วยจัดการด้านความเร็วและความแม่นยำในการค้นหา (Relevancy)

## 🏗️ สถาปัตยกรรมระดับลึก (In-depth Architecture)

### 1. Inverted Index Concept
หัวใจของ Elasticsearch คือการเก็บข้อมูลแบบ **Inverted Index** แทนที่จะเก็บเป็นแถวเหมือน Database ทั่วไป มันจะแยก "คำ" (Tokens) ออกมาแล้วเก็บลิสต์ว่าคำนี้ปรากฏอยู่ที่เอกสารใบไหนบ้าง ทำให้การค้นหาทำได้รวดเร็วระดับมิลลิวินาทีไม่ว่าข้อมูลจะเยอะแค่ไหน

### 2. Thai Analysis Pipeline
ภาษาไทยมีความยากตรงที่ "ไม่มีการเว้นวรรคระหว่างคำ" เราจึงต้องใช้ **Thai Tokenizer** (เช่น `icu_analyzer` หรือ `thai`) ในการตัดคำให้ถูกต้อง
* **Tokenization:** "ภาษาไทยง่ายมาก" -> ["ภาษา", "ไทย", "ง่าย", "มาก"]
* **Synonyms:** ตั้งค่าให้ "มือถือ" และ "โทรศัพท์" มีความหมายเดียวกันในการค้นหา

---

## 🧪 Lab: Thai Search & Fuzzy Logic

Lab นี้จะจำลองการตั้งค่า Index ที่รองรับภาษาไทยและค้นหาคำที่พิมพ์ผิด (Fuzzy Search)

### 🟩 Step 1: สร้าง Index พร้อม Thai Analyzer
```json
PUT /jobs_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "thai_worker": {
          "tokenizer": "thai",
          "filter": ["lowercase", "decimal_digit"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "job_title": {
        "type": "text",
        "analyzer": "thai_worker"
      }
    }
  }
}
```
🟧 Step 2: การค้นหาแบบ Fuzzy (รองรับการพิมพ์ผิด)
```JSON
GET /jobs_index/_search
{
  "query": {
    "match": {
      "job_title": {
        "query": "วิศวะกร", 
        "fuzziness": "AUTO",
        "prefix_length": 1
      }
    }
  }
}
```
แม้ผู้ใช้จะพิมพ์ผิดจาก "วิศวกร" เป็น "วิศวะกร" ระบบก็ยังหาเจอ
---
## ⚠️ ข้อควรระวังและการจัดการปัญหา (Gotchas & Pitfalls)
การรัน Search Engine ในระดับ Production มีความซับซ้อนที่ต้องระวังดังนี้:

1. ปัญหา "Brain Split" และ Resource Management
ปัญหา: Elasticsearch กิน RAM มหาศาล (Heap Memory) หากตั้งค่าไม่เหมาะสมหรือรัน Node เดียวแล้วล่ม อาจทำให้ข้อมูลพังหรือค้นหาไม่ได้

วิธีแก้: ต้องรันแบบ Cluster (อย่างน้อย 3 Nodes) และตั้งค่า Heap Size ให้เหมาะสม (แนะนำครึ่งหนึ่งของ RAM เครื่องแต่ไม่เกิน 32GB) และใช้ Shard Strategy ที่ถูกต้องตามปริมาณข้อมูล

2. ความท้าทายในการตัดคำภาษาไทย (Segmentation Ambiguity)
ปัญหา: Tokenizer พื้นฐานอาจตัดคำผิดความหมาย เช่น "ตากลม" อาจตัดเป็น "ตา-กลม" (Eyes) หรือ "ตาก-ลม" (Dry in wind)

วิธีแก้: ใช้ Custom Dictionary เพื่อเพิ่มคำศัพท์เฉพาะทาง (เช่น ชื่อตำแหน่งงานเฉพาะ) เข้าไปใน Analyzer เพื่อให้ระบบตัดคำได้แม่นยำขึ้นตามบริบทของแอปฯ

3. ข้อมูลใน Search ไม่ตรงกับ Database (Data Sync Lag)
ปัญหา: เมื่อ Update ข้อมูลใน SQL แต่ข้อมูลใน Elasticsearch ไม่เปลี่ยนตามทันที (Near Real-time) ทำให้ User ค้นหาแล้วเจอข้อมูลเก่า

วิธีแก้: ใช้แนวทาง CDC (Change Data Capture) เช่น Debezium หรือใช้คิว (RabbitMQ/Kafka) เพื่อส่งสัญญาณให้ Elasticsearch อัปเดตข้อมูลทันทีหลังจากที่ Database หลักมีการเปลี่ยนแปลง

4. ปัญหาการทำ Aggregations บนฟิลด์ Text
ปัญหา: การสั่งสรุปผล (เช่น นับจำนวนงานแยกตามทำเล) บนฟิลด์ที่เป็น text จะทำให้กิน Memory สูงมากจน Circuit Breaker อาจสั่งหยุดทำงาน

วิธีแก้: ต้องใช้ฟิลด์ประเภท keyword สำหรับข้อมูลที่ต้องการทำสรุปผล (Aggregation/Sorting) และใช้ text สำหรับการค้นหา (Full-text Search) เท่านั้น
