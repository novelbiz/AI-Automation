# 🤖 Chatbot RAG to LINE  
> สร้าง AI Chatbot เชื่อมต่อ LINE ด้วย n8n + Gemini + Supabase (RAG)

---

🎥 วิดีโอสอน: 
[![Watch the video](https://github.com/novelbiz/AI_Automation/blob/main/assets/thumbnail/RAG%20%2B%20LINE.png)](https://youtu.be/0N3X9lG6n2c?si=Hg9EALEiRzab790s)

---

## 🧠 หลักการทำงาน (RAG Workflow)

1. รับข้อความจาก LINE ผ่าน Webhook  
2. AI Agent วิเคราะห์คำถาม  
3. ดึงข้อมูลจาก Supabase Vector Store  
4. ใช้ Google Gemini ประมวลผลคำตอบตามบริบท  
5. แปลงข้อความเป็น Flex Message  
6. ตอบกลับผู้ใช้ใน LINE แบบสวยงามและสุภาพ  
---

## 🔐 สิ่งที่ต้องเตรียมก่อนใช้งาน

| รายการ | รายละเอียด |
|--------|-------------|
| 🔗 LINE Channel Access Token | ใช้ใน HTTP Request สำหรับตอบกลับ |
| 🧠 Gemini API Key | สำหรับโมเดลภาษา Google Gemini |
| 🗃️ Supabase Project | มี table `documents` สำหรับเก็บข้อมูล |
| 🤖 Ollama Server | ใช้สร้าง Embedding สำหรับค้นหาเวกเตอร์ |

---
## ⚙️ Config Workflow

### 1. **Webhook**
- จุดเริ่มต้นของ workflow  
- รับข้อความจากผู้ใช้ใน LINE (HTTP `POST`)  

---

### 2. **Edit Fields (Set Node)**
- ดึงค่าที่จำเป็นจาก Webhook:
  - `body.events[0].message.text` → ข้อความจากผู้ใช้  
  - `body.events[0].replyToken` → token สำหรับตอบกลับ  

---

### 3. **AI Agent**
- เป็นหัวใจหลักของระบบ  
- Prompt ระบบ (System Prompt):
  - คุณคือผู้ช่วยตอบแชทอย่างเป็นทางการใน LINE ของร้านกาแฟ ☕  
  - ห้ามแต่งเติมข้อมูลเกินจริง  
  - ใช้ข้อมูลจากฐาน Supabase Vector Store เป็นหลัก  
- ใช้โมเดล Gemini และ RAG เชื่อม Supabase เพื่อค้นข้อมูลก่อนตอบ  

> **กฎการตอบสำคัญ**
> - ❌ ห้ามเดา  
> - ✅ ถ้าไม่พบข้อมูล → ตอบ “ขอทวนคำถามอีกครั้งได้ไหมครับ ร้านของเราคือร้านกาแฟ”  
> - ❓ ถ้าไม่เกี่ยวข้อง → “ขออภัยในความไม่สะดวกครับ”

---

### 4. **Supabase Vector Store**
- ใช้ฐานข้อมูล `documents` จาก Supabase  
- ทำหน้าที่ค้นหาข้อมูล (Vector Retrieval)  
- ใช้ Embedding จาก **Ollama** เพื่อเทียบข้อความที่คล้ายกัน  

---

### 5. **Embeddings Ollama**
- สร้างเวกเตอร์ฝังข้อความด้วยโมเดล `nomic-embed-text:latest`  
- ใช้เพื่อประมวลผลข้อมูลที่เกี่ยวข้องกับ RAG  

---

### 6. **Code Node**
- แปลงข้อความผลลัพธ์จาก AI ให้เป็นโครงสร้าง **LINE Flex Message**  
- ฟังก์ชันหลัก:
  - ทำความสะอาดข้อความ (ลบสัญลักษณ์ไม่จำเป็น)
  - แยกบรรทัดข้อความให้อ่านง่าย
  - สร้าง array ของ text objects สำหรับ Flex Bubble

---

### 7. **HTTP Request**
- ส่งคำตอบกลับไปยัง LINE API  
- Endpoint: `https://api.line.me/v2/bot/message/reply`  
- Header:
  - `Content-Type: application/json`
  - `Authorization: Bearer <LINE_CHANNEL_ACCESS_TOKEN>`
- รูปแบบข้อความ: **Flex Message**  
  แสดงหัวข้อ “📌 ข้อมูลระบบ” พร้อมเนื้อหาที่ได้จาก AI

---

### 8. **Google Gemini Chat Model 2**
- ใช้โมเดลภาษา **Google Gemini (PaLM)**  
- ทำงานคู่กับ AI Agent เพื่อสร้างคำตอบภาษาธรรมชาติ  
- เป็นส่วนประมวลผลภาษาหลักของ Chatbot

---

## 🔄 ลำดับการทำงาน (Workflow Flow)

| ลำดับ | Node | หน้าที่หลัก | การเชื่อมต่อถัดไป |
|--------|------|--------------|--------------------|
| 1 | **Webhook** | รับข้อความจากผู้ใช้ LINE ผ่าน HTTP POST | ส่งต่อไปที่ **Edit Fields** |
| 2 | **Edit Fields** | แยกข้อมูลสำคัญ เช่น ข้อความผู้ใช้และ replyToken | ส่งต่อไปที่ **AI Agent** |
| 3 | **AI Agent** | วิเคราะห์ข้อความ + ใช้โมเดล Gemini และ RAG | ส่งต่อไปยัง **Code Node** และเรียกใช้:<br>• **Supabase Vector Store** (RAG Tool)<br>• **Google Gemini Chat Model 2** (Language Model) |
| 4 | **Supabase Vector Store** | ดึงข้อมูลจากฐาน Supabase เพื่อให้ AI ใช้ตอบ | ใช้ Embedding จาก **Embeddings Ollama** |
| 5 | **Embeddings Ollama** | แปลงข้อความเป็นเวกเตอร์เพื่อใช้ค้นหาข้อมูลคล้ายกัน | ส่งเวกเตอร์ให้ **Supabase Vector Store** |
| 6 | **Code Node** | จัดรูปแบบข้อความ AI ให้อยู่ในรูป Flex Message ของ LINE | ส่งต่อไปที่ **HTTP Request** |
| 7 | **HTTP Request** | ส่งข้อความตอบกลับไปยัง LINE Bot API | จบการทำงานของ Workflow |


---

## 📌 หมายเหตุ

- Workflow นี้ออกแบบสำหรับร้านกาแฟ สามารถปรับ prompt และ vector store ได้ตามธุรกิจ  
- ห้ามเปิดเผย token หรือ API key ในไฟล์สาธารณะ  
- เหมาะสำหรับ chatbot ที่ต้องตอบข้อมูลจากฐานจริง (ไม่เดา/ไม่แต่งเติม)

---

### 💬 ตัวอย่างข้อความตอบกลับใน LINE
```
📌 ข้อมูลระบบ
- ร้านเปิดทุกวัน เวลา 8.00–17.00 น. ครับ
- มีบริการเครื่องดื่มและเบเกอรี่โฮมเมดครับ
ℹ️ หากต้องการรายละเอียดเพิ่มเติม กรุณาติดต่อฝ่ายสนับสนุน
```

