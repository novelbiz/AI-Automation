
# 🧠 RAG Gemini + Ollama + Supabase Vector Workflow (n8n)

Workflow นี้ออกแบบเพื่อกระบวนการ **RAG (Retrieval-Augmented Generation)** ด้วยการนำเอกสารจาก Google Drive มาประมวลผล, สร้าง Metadata, แบ่งเป็น Chunks, ฝังด้วย **Ollama**, จัดเก็บลง **Supabase Vector Store**, และตั้งค่า Agent สำหรับตอบคำถามผ่าน **Google Gemini** + **Ollama** โดยรองรับ LINE Messaging API

---

🎥 วิดีโอสอน: 
[![Watch the video](https://github.com/novelbiz/AI_Automation/blob/main/episodes/EP07_RAG_Gemini_Ollama_Supabase_n8n/Image/Rag%20Gemini%20Ollama%20Supabase%20Vector.png)](https://youtu.be/cj5foG9ATXg?si=73Td80vfwU9higDz)

---

## ⚙️ ส่วนประกอบหลักและ Credentials

| บริการ | Credential Type | Placeholder ใน Workflow |
| :--- | :--- | :--- |
| Google Drive | OAuth2 / API Key | `PLACEHOLDER_GOOGLE_DRIVE_CREDENTIAL_ID` |
| Google Gemini (PaLM) | API Key | `PLACEHOLDER_GEMINI_CREDENTIAL_ID` |
| Ollama | API Key / Endpoint | `PLACEHOLDER_OLLAMA_CREDENTIAL_ID` |
| Supabase | API Key | `PLACEHOLDER_SUPABASE_CREDENTIAL_ID` |
| LINE Messaging API | Access Token | `PLACEHOLDER_LINE_ACCESS_TOKEN` |

---

## 🚀 Node Workflow รายละเอียดสมบูรณ์

### #Node: Document Data
```json
{
  "data":  {{ JSON.stringify($json.text) }}
}
```

### #Node: AI Agent (แก้ไขคำผิด)
**Prompt (User Message):**
```markdown
กรุณาช่วยตรวจสอบและแก้ไขคำผิดในข้อความต่อไปนี้ให้ถูกต้องตามหลักภาษาไทย โดยไม่เปลี่ยนความหมายของเนื้อหา หากมีคำสะกดผิดหรือใช้คำไม่เหมาะสม ให้แก้ไขให้เหมาะสม: {{ $json.data }}
ไม่ต้องสรุปการแก้ไขให้ดู
```

### #Node: Create Metadata Title & Description
**Prompt (User Message):**
```markdown
โปรดสร้าง title และ description โดยอิงจากเนื้อหาเอกสารที่กำหนดไว้ด้านล่างนี้ จากข้อความนี้: {{ $json.output }}
ข้อมูลที่สร้างจะใช้สำหรับแยกข้อมูลในฐานข้อมูลเวกเตอร์ (vector DB) เพื่อการค้นคืนแบบ RAG

**กรุณาตอบกลับเป็นภาษาไทยเท่านั้น**
**ไม่ต้องเพิ่มคำอธิบายใดๆ นอกเหนือจาก metadata title และ metadata description**
```

### #Node: Structured Output Parser (JSON Example)
```json
[
  {
    "title": "Test Title 1 (Replace it with real title)",
    "description": "Test Description 1 (Replace it with real description)"
  },
  {
    "title": "Test Title 2 (Replace it with real title)",
    "description": "Test Description 2 (Replace it with real description)"
  },
  {
    "title": "Test Title 3 (Replace it with real title)",
    "description": "Test Description 3 (Replace it with real description)"
  }
]
```

### #Node: Code (Split into Chunks)
```javascript
const chunkSize = 1000;
const chunkOverlap = 200;

// รับข้อความจาก Node ก่อนหน้า
const rawText = $('AI Agent').first().json.output;

// ล้างข้อความ: ลบหมายเหตุในวงเล็บที่มีคำว่า "ปรับ"
const cleanedText = rawText.replace(/\([^)]*?ปรับ[^)]*?\)/g, "").trim();

const chunks = [];
let remainingText = cleanedText;

while (remainingText.length > 0) {
    let splitPoint = remainingText.lastIndexOf("

", chunkSize);
    if (splitPoint === -1) splitPoint = remainingText.lastIndexOf(". ", chunkSize);
    if (splitPoint === -1) splitPoint = remainingText.lastIndexOf(" ", chunkSize);
    if (splitPoint === -1 || splitPoint < chunkSize * 0.5) splitPoint = chunkSize;

    const chunk = remainingText.substring(0, splitPoint).trim();
    chunks.push(chunk);

    remainingText = remainingText.substring(Math.max(0, splitPoint - chunkOverlap)).trim();

    if (remainingText.length < chunkSize * 0.2) {
        chunks.push(remainingText);
        break;
    }
}

// ส่งออกแบบ array ตาม format ที่ Cohere Embed ต้องการ
return [{
  json: {
    texts: chunks,
    input_type: "search_document"
  }
}];
```

### #Node: Basic LLM Chain (Process Context)
```markdown
<document> 
{{ $('AI Agent').item.json.output }}
</document> 

<chunk> 
{{ $('Split Out').item.json.texts }}
</chunk> 

โปรดดำเนินการดังต่อไปนี้:
- สรุปและเขียนบริบท (context) สั้น ๆ เพื่ออธิบายว่า chunk ด้านล่างนี้อยู่ในส่วนใดของเอกสาร เพื่อช่วยในการค้นหาและดึงข้อมูลได้ดีขึ้น
- คืนค่า chunk ดั้งเดิมเหมือนเดิมทุกประการ **เว้นแต่จำเป็นต้องแก้ไข**
- หาก chunk มีตัวเลข เปอร์เซ็นต์ หรือชื่อหน่วยงานไม่สมบูรณ์ ให้ใช้ข้อมูลจากเอกสารฉบับเต็มในการเติมให้ครบ
- หากเป็นเพียงบางส่วนของประโยคที่ถูกตัดออก ให้เติมคำที่หายไปเฉพาะในกรณีจำเป็นต่อความเข้าใจ
- หาก chunk อยู่ในตาราง ให้แสดงทั้งแถวของตารางเพื่อรักษาความสมบูรณ์ของข้อมูล

**รูปแบบการตอบกลับ:**
[บริบทโดยย่อ] : [chunk ดั้งเดิม หรือฉบับแก้ไขถ้าจำเป็น]

**คำตอบต้องอยู่ในภาษาไทยเท่านั้น** และ **อย่าเพิ่มคำอธิบายหรือรูปแบบใดๆ นอกเหนือจากที่ระบุ**
```

### #Node: AI Agent (Chat)
✅ **Prompt (User Message)**
```markdown
ต่อไปนี้คือคำถามจากผู้ใช้:  
{{ $json.chatInput }}

ขั้นตอนที่คุณต้องดำเนินการมีดังนี้:
1. วิเคราะห์คำถาม และปรับให้กระชับ ชัดเจน เหมาะสำหรับการใช้ค้นหาข้อมูลจากเวกเตอร์ดาต้าเบส (เช่น ตัดคำฟุ่มเฟือย หรือคำเกริ่น)
2. ใช้คำถามที่ปรับแล้วเพื่อค้นหาข้อมูลจาก `Supabase Vector Store` ซึ่งเป็นเวกเตอร์ดาต้าเบสภายในองค์กร
3. ตอบคำถามโดยอ้างอิงจากเนื้อหาที่ค้นเจอ **เท่านั้น**

ห้ามเดา ห้ามสร้างเนื้อหาที่ไม่มีอยู่จริง  
หากไม่พบข้อมูลที่เกี่ยวข้อง ให้ตอบว่า:  
"นั่นไม่ใช่คำที่มีอยู่ในฐานข้อมูล"  
หากพบข้อมูลบางส่วนแต่ยังไม่ชัดเจน โปรดแจ้งให้ผู้ใช้เพิ่มคำค้นหรือบริบทเพิ่มเติม

**คืนค่าคำตอบสุดท้ายเท่านั้น**
```

✅ **System Message**
```markdown
คุณคือผู้ช่วยภายในองค์กร ที่สามารถตอบคำถามได้โดยอิงจากข้อมูลในระบบเวกเตอร์ดาต้าเบสเท่านั้น

คุณมีเครื่องมือชื่อว่า "supabase vector store" ซึ่งใช้สำหรับค้นหาข้อมูลจากเอกสารภายในองค์กรที่ถูกฝัง (embedding) ไว้ใน Supabase Vector Store

**คำแนะนำ:**
- ห้ามใช้ความรู้ทั่วไปของคุณในการตอบคำถาม
- ต้องเรียกใช้เครื่องมือ `supabase vector store` ทุกครั้งก่อนจะตอบคำถาม
- ถ้าผลลัพธ์จากเครื่องมือไม่มีข้อมูลที่เกี่ยวข้อง (`results` ว่างเปล่า) ให้ตอบว่า:  
  "นั่นไม่ใช่คำที่มีอยู่ในฐานข้อมูล"
- ห้ามเดา ห้ามแต่งเรื่อง และห้ามให้ข้อมูลจากแหล่งอื่นนอกเหนือจากที่เครื่องมือส่งมา

กรุณาตอบกลับเป็นข้อความภาษาธรรมชาติที่อ่านง่าย ไม่แสดง JSON, โค้ด หรือโครงสร้างข้อมูลดิบใดๆ
```

---
