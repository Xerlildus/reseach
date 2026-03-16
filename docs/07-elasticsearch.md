## 🔍 7. Elasticsearch Mastery
การทำ Search ที่รองรับภาษาไทย (Thai Tokenizer)
* **Inverted Index:** ทำความเข้าใจการเก็บข้อมูลแบบดัชนีเพื่อให้ Search เร็วระดับมิลลิวินาที
* **Fuzzy Search:** ตั้งค่าให้ระบบค้นหาคำที่ใกล้เคียงได้แม้ผู้ใช้จะพิมพ์ผิด (เช่น "โทสับ" -> "โทรศัพท์")
* **Example:** **Job Portal** - ค้นหาตำแหน่งงานจาก Keyword, ทำเล และช่วงเงินเดือน พร้อมสรุปจำนวนงานแต่ละประเภท (Aggregations) ทันที
```JSON
// Fuzzy Search สำหรับค้นหาคำใกล้เคียง
{
  "query": {
    "match": {
      "job_title": {
        "query": "โทสับ",
        "fuzziness": "AUTO"
      }
    }
  }
}
```
