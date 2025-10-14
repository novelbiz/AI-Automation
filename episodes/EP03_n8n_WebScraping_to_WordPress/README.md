# EP03_n8n_WebScraping_to_WordPress

> Workflow ตัวอย่าง: **ดึงข้อมูลจากเว็บไซต์ → ประมวลผลด้วย AI → โพสต์ลง WordPress อัตโนมัติ**  
> ใช้ n8n + Google Gemini / AI Agent เพื่อสร้างบทความในรูปแบบ HTML ที่พร้อมโพสต์ทันที

🎥 วิดีโอสอน: [EP03 n8n WebScraping to WordPress](https://www.youtube.com/watch?v=M_UnuPLWCfk)

---

## 🎯 เป้าหมายของ Workflow

1. ดึงข้อมูลจากเว็บไซต์เป้าหมาย (เช่น ข่าว, บทความ, ภาพ)
2. สร้างบทความใหม่ด้วย AI ตามแนวทาง SEO / สไตล์ที่กำหนด
3. อัปโหลดภาพไปยัง WordPress Media Library
4. สร้างโพสต์ใหม่ใน WordPress และตั้ง Featured Image อัตโนมัติ

---

## ⚙️ API Endpoint ที่ใช้ใน Workflow

### 🖼️ Upload Image to WordPress
```bash
POST https://www.example.com/wp-json/wp/v2/media
```

### 📝 Post to WordPress
```bash
POST https://www.example.com/wp-json/wp/v2/posts
```

### ⭐ Update Featured Media to WordPress
```bash
POST https://www.example.com/wp-json/wp/v2/posts/{{ $json.id }}
```

> 🧩 หมายเหตุ:  
> อย่าลืมเปลี่ยน `https://www.example.com` ให้เป็นโดเมนจริงของคุณ  
> และตรวจสอบว่า credentials (Basic Auth / JWT Token) ถูกต้อง  

---

## 🤖 AI Agent Prompt (User Message)

```text
เขียนบทความเกี่ยวกับ {{ $json.post[0] }} แปลเป็นภาษาไทยก่อน โดยเน้นไปที่ AI พร้อมคำแนะนำสำหรับผู้อ่าน สร้างหัวข้อย่อยและย่อหน้าที่มีความสอดคล้องกับเนื้อหา ใช้การจัดรูปแบบ HTML สำหรับการแสดงผลบน WordPress ด้วยการใช้แท็กที่เหมาะสม เช่น <h1>, <h2>, <p>, <ul>, <li>, <strong>, และ <em>"

คำแนะนำในการเขียน:
- ต้องมีเนื้อหาที่ลึกซึ้งและน่าสนใจ  
- ใช้หัวข้อหลักและย่อยเพื่อแยกเนื้อหาให้เข้าใจง่าย  
- ใส่คำสำคัญหรือข้อความที่เน้นด้วย <strong> และ <em> หากจำเป็น  
- สร้างรายการหรือข้อแนะนำที่มีความหมายด้วย <ul> และ <li>  
- สรุปท้ายบทความด้วยข้อคิดเห็นหรือคำแนะนำ  

โปรดสร้างบทความออกมาในรูปแบบ HTML โดยไม่มีการใช้ข้อความเกินความจำเป็น  

ข้อกำหนดสำคัญ:
- ห้ามคัดลอกหัวข้อข่าวมาโดยตรง ต้องเขียนใหม่  
- ต้องเขียนให้เหมาะกับแนวทางของแต่ละ Platform  
- โพสต์ต้องมีอารมณ์เชิงบวก หรือเป็นกลางเสมอ  
```

---

## 🧱 Structured Output Parser

ใช้เพื่อกำหนดรูปแบบข้อมูลที่ AI จะต้องส่งกลับให้ตรงกับ workflow ที่ n8n รองรับ

```json
{
  "type": "object",
  "properties": {
    "name": { "type": "string" },
    "description": { "type": "string" },
    "platform_posts": {
      "type": "object",
      "properties": {
        "wordpress": {
          "type": "object",
          "properties": {
            "Title": { "type": "string" },
            "post": { "type": "string" },
            "hashtags": {
              "type": "array",
              "items": { "type": "string" }
            }
          },
          "additionalProperties": false
        }
      },
      "additionalProperties": true
    },
    "additional_notes": { "type": "string" }
  },
  "additionalProperties": false
}
```

---

## 🔩 ขั้นตอนใช้งาน Workflow

1. **Import Workflow**  
   - ดาวน์โหลดไฟล์ `workflow.json`  
   - ไปที่ n8n → Import from File  

2. **ตั้งค่า Credentials**
   - WordPress (HTTP Basic Auth หรือ JWT)  
   - AI Agent (Google Gemini API Key)  

3. **แก้ไข Endpoint และ Prompt**
   - แก้ URL ให้ตรงกับ WordPress ของคุณ  
   - ปรับ prompt ให้เข้ากับธีมเนื้อหาที่ต้องการ  

4. **ทดสอบรันครั้งแรก**
   - ลองรันแบบ manual ก่อนตั้ง schedule  
   - ตรวจสอบว่าโพสต์ขึ้นจริงใน WordPress  

5. **ตั้ง Schedule Trigger**
   - ถ้าอยากให้โพสต์อัตโนมัติเป็นรายวัน/สัปดาห์ ให้ตั้งใน n8n  

---

## 🛡️ ความปลอดภัย

- Workflow นี้ **ไม่มี API key หรือ credential id** ถูกลบออกแล้วเพื่อความปลอดภัย  
- ต้องเพิ่ม credential ใหม่ใน n8n ก่อนใช้งานจริง  
- ห้ามเผยแพร่ API key หรือ password บน GitHub หรือสาธารณะ  

---

## 💡 หมายเหตุเพิ่มเติม

- AI Agent สามารถปรับให้เขียนบทความแนว SEO, ข่าวเทคโนโลยี, หรือ How-to ได้  
- หาก WordPress ใช้ plugin security เช่น JWT Auth ต้องแน่ใจว่า endpoint เปิดใช้งานอยู่  
- แนะนำให้ทดสอบ API แต่ละจุดด้วย Postman ก่อน เพื่อดู response structure  

---

## 🧠 สรุป

Workflow นี้เป็นจุดเริ่มต้นที่ยอดเยี่ยมสำหรับการ **เชื่อม AI เข้ากับระบบ CMS อย่าง WordPress**  
ทำให้คุณสามารถสร้างเนื้อหาอัตโนมัติที่มีคุณภาพ เหมาะสำหรับสายคอนเทนต์ มาร์เก็ตติ้ง หรือทีมข่าวที่ต้องการระบบโพสต์อัจฉริยะ 🚀
