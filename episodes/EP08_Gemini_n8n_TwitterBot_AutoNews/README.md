# 🤖 N8N + Google Gemini: ระบบสรุปข่าวและโพสต์ลง Twitter (X) อัตโนมัติ

เอกสารนี้แสดงรายละเอียด Workflow สำหรับสร้างบอทอัตโนมัติที่ดึงข้อมูลข่าวจากเว็บไซต์ที่กำหนด (Web Scraping) สรุปเนื้อหาด้วย **Google Gemini** และโพสต์เป็นข้อความพร้อมรูปภาพไปยังบัญชี Twitter/X ของผู้ใช้งานโดยอัตโนมัติ

---

🎥 วิดีโอสอน: 

[![Watch the video](https://github.com/novelbiz/AI_Automation/blob/main/episodes/EP08_Gemini_n8n_TwitterBot_AutoNews/Image/Gemini_n8n_TwitterBot_AutoNews.png)](https://youtu.be/7sWdr6eivzc?si=QEeYKND2MZRYe5_J)

---

## ขั้นตอนการตั้งค่า Node

   * Schedule Trigger (ตั้งเวลาการทำงาน)
   * Edit Fields (กำหนด URL เว็บไซต์)
   * HTTP Request (ดึง HTML)
   * HTML (แยกข้อมูล: รูปภาพและหัวข้อข่าว)
   * Aggregate (รวมข้อมูล)
   * Google Gemini Chat Model (ตั้งค่า Credential)
   * Basic LLM Chain (สร้าง Prompt สรุปข่าว)
   * HTTP Request1 (ดึงรูปภาพ)
   * Edit Image (ปรับขนาดรูปภาพ)
   * HTTP Request2 (อัปโหลด Media ไปยัง Twitter/X)
   * Code (ทำความสะอาดข้อความ)
   * X (โพสต์ Tweet/X)

## 1. ภาพรวม Workflow

Workflow จะดำเนินการตามลำดับดังนี้:

| ลำดับ | Node             | หน้าที่                                                             |
| :---- | :--------------- | :------------------------------------------------------------------ |
| 1     | Schedule Trigger | กำหนดเวลาให้ Workflow ทำงานอัตโนมัติ (เช่น 09:00 น.)                |
| 2     | Edit Fields      | กำหนด URL ของเว็บไซต์ข่าวที่ต้องการ                                 |
| 3     | HTTP Request     | ดึง HTML ของหน้าเว็บ                                                |
| 4     | HTML             | แยก URL รูปภาพและหัวข้อข่าวจาก HTML                                 |
| 5     | Aggregate        | รวมหัวข้อข่าวและ URL รูปภาพที่เลือก (เช่น `post[0]` และ `image[2]`) |
| 6     | Basic LLM Chain  | ส่งหัวข้อข่าวให้ Gemini สรุปและสร้าง Content สำหรับ Twitter/X       |
| 7     | HTTP Request1    | ดึงไฟล์รูปภาพจาก URL ที่ได้                                         |
| 8     | Edit Image       | ปรับขนาดรูปภาพให้เหมาะสมก่อนอัปโหลด (แนะนำ 500x500 พิกเซล)          |
| 9     | HTTP Request2    | อัปโหลดรูปภาพไปยัง Twitter/X (จำเป็นต้องใช้ OAuth 1.0)              |
| 10    | Code             | ทำความสะอาดข้อความที่ได้จาก Gemini (ลบ `\n`, `#`, `*`)              |
| 11    | X                | โพสต์ข้อความและ Media ID ลงใน Twitter/X (จำเป็นต้องใช้ OAuth 2.0)   |
---
## 2. ขั้นตอนการตั้งค่า Node

### 2.1 Schedule Trigger

* **Node Name:** Schedule Trigger
* **Parameters:**

  * `Rule`: `Interval`
  * `Trigger At Hour`: `9` (หรือเวลาที่ต้องการให้ Workflow รัน)

### 2.2 Edit Fields

* **Node Name:** Edit Fields
* **Parameters:**

  * `Operation`: `Set`
  * `Assignments / Name`: `url`
  * `Assignments / Value`: URL ของเว็บไซต์ข่าวที่ต้องการ (เช่น `https://www.artificialintelligence-news.com/`)

### 2.3 HTTP Request (ดึง HTML)

* **Node Name:** HTTP Request
* **Parameters:**

  * `Method`: `GET`
  * `URL`: `={{ $json.url }}`
  * `Options / Response / Response Format`: `Text`

### 2.4 HTML (แยกข้อมูล: รูปภาพและหัวข้อข่าว)

* **Node Name:** HTML
* **Parameters:**

  * `Operation`: `Extract Html Content`
  * **Extraction Values**

    * `image` (รูปภาพ):

      * `CSS Selector`: `div img`
      * `Return Value`: `Attribute` → `src`
      * `Return Array`: เปิด
    * `post` (หัวข้อข่าว):

      * `CSS Selector`: `h1 a`
      * `Return Array`: เปิด

### 2.5 Aggregate (รวมข้อมูล)

* **Node Name:** Aggregate
* **Parameters:**

  * `Fields To Aggregate`:

    * `Field 1`: `post[0]` (หัวข้อข่าวแรก)
    * `Field 2`: `image[2]` (URL รูปภาพที่เลือก)

### 2.6 Google Gemini Chat Model

* **Node Name:** Google Gemini Chat Model
* **Credentials:** ต้องสร้าง **Google Gemini (PaLM) API account** Credential
* **Parameters:**

  * `Model Name`: `models/gemini-2.0-flash` (หรือเวอร์ชัน 2 ขึ้นไป)

### 2.7 Basic LLM Chain (สร้าง Prompt สรุปข่าว)

* **Node Name:** Basic LLM Chain
* **Parameters:**

  * `Source for Prompt`: `Define Below`
  * `Prompt User Message`:

    ```markdown
    =เขียนบทความใหม่เกี่ยวกับ {{ $json.post[0] }} ไม่เกิน 200 ตัวอักษร โพสต์ลง Twitter

    ข้อกำหนดสำคัญ:
    - ห้ามคัดลอกหัวข้อข่าวมาโดยตรง
    - ต้องเขียนใหม่ให้เหมาะสมกับแต่ละ Platform
    - โพสต์ต้องมีอารมณ์เชิงบวกหรือเป็นกลาง
    ```
  * *หมายเหตุ:* `{{ $json.post[0] }}` คือหัวข้อข่าวจาก Node Aggregate

### 2.8 HTTP Request1 (ดึงรูปภาพ)

* **Node Name:** HTTP Request1
* **Parameters:**

  * `Method`: `GET`
  * `URL`: `={{ $('Aggregate').item.json.image[2][0] }}`

### 2.9 Edit Image (ปรับขนาดรูปภาพ)

* **Node Name:** Edit Image
* **Parameters:**

  * `Operation`: `Resize`
  * `Width`: `500`
  * `Height`: `500`

### 2.10 HTTP Request2 (อัปโหลด Media ไปยัง Twitter/X)

* **Node Name:** HTTP Request2
* **Credentials:** ต้องสร้าง **Twitter (OAuth1)** Credential
* **Parameters:**

  * `Method`: `POST`
  * `URL`: `https://upload.twitter.com/1.1/media/upload.json`
  * `Authentication`: `Generic Credential Type`
  * `Generic Auth Type`: `OAuth1 Api`
  * `Body Content Type`: `Multipart/Form-Data`
  * `Body Parameters`:

    * `Parameter Type`: `n8n Binary File`
    * `Name`: `media`
    * `Input Data Field Name`: `data`

### 2.11 Code (ทำความสะอาดข้อความ)

* **Node Name:** Code
* **Parameters:**

  ```javascript
  return items.map(item => {
    const output = $('Basic LLM Chain').first().json.text || "";
    const cleanedOutput = output
      .replace(/\n/g, "")
      .replace(/\#/g, "")
      .replace(/\*/g, "");
    return {
      json: {
        ...item.json,
        output: cleanedOutput
      }
    };
  });
  ```

### 2.12 X (โพสต์ Tweet/X)

* **Node Name:** X
* **Credentials:** ต้องสร้าง **X account 2 (OAuth2)** Credential
* **Parameters:**

  * `Operation`: `Create Tweet`
  * `Text`: `={{ $json.output }}`
  * `Additional Fields / Attachments`: `={{ $json.media_id_string }}`
---
## 3. 💡 Tips & Best Practices

1. **ตั้งเวลา Workflow ให้เหมาะสม** – เลือกช่วงเวลาที่ผู้ใช้งาน Twitter/X มีความเคลื่อนไหวสูง เช่น ช่วงเช้า 08:00-10:00 หรือช่วงเย็น 17:00-19:00 เพื่อเพิ่มการเข้าถึงโพสต์

2. **ตรวจสอบ URL ข่าวเป็นประจำ** – หากเว็บไซต์มีการเปลี่ยนโครงสร้าง HTML, CSS Selector ที่ตั้งไว้ใน Node HTML อาจต้องปรับให้สอดคล้องกับโครงสร้างใหม่

3. **จำกัดความยาวข้อความ** – แม้ Google Gemini จะสรุปข่าวได้ดี ควรจำกัดความยาวข้อความไม่เกิน 200 ตัวอักษร เพื่อให้เหมาะสมกับรูปแบบโพสต์ของ Twitter/X

4. **ปรับขนาดรูปภาพอย่างเหมาะสม** – แนะนำขนาด 500x500 พิกเซลเพื่อให้แน่ใจว่ารูปภาพไม่ถูกครอปหรือผิดสัดส่วนเมื่อแสดงในฟีด

5. **ใช้ OAuth Credential อย่างระมัดระวัง** – เก็บข้อมูล OAuth อย่างปลอดภัย และทดสอบ Credential ก่อนใช้งานจริงเพื่อหลีกเลี่ยงการล้มเหลวในการโพสต์

6. **ทดสอบ Workflow เป็นระยะ** – ก่อนเปิดใช้งานเต็มรูปแบบ ให้รัน Workflow ด้วยข้อมูลตัวอย่างเพื่อตรวจสอบการทำงานของ Node ทุกตัว และแก้ไขข้อผิดพลาดล่วงหน้า

7. **ตรวจสอบข้อความก่อนโพสต์** – หาก Workflow โพสต์อัตโนมัติ ควรมี Node ตัวกลางสำหรับตรวจสอบข้อความสำคัญหรือคีย์เวิร์ดที่ไม่เหมาะสม เพื่อหลีกเลี่ยงข้อผิดพลาดหรือเนื้อหาที่ไม่ต้องการ

