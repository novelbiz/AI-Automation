# ☕ เชื่อมต่อ Supabase Vector Store กับ n8n ทำ RAG Workflow ง่าย ๆ
---

🎥 วิดีโอสอน: 
[![Watch the video](https://github.com/novelbiz/AI_Automation/blob/main/assets/thumbnail/classify%20rag.png)](https://youtu.be/Uri9cQHIks0?si=_6qzJ84peRPHgr0S)

---
## 🧠 ภาพรวม
Workflow นี้ถูกออกแบบให้ทำงานเป็นระบบ **AI Chatbot สำหรับร้านกาแฟใน LINE Official Account** โดยมีหน้าที่ดังนี้:

1. รับข้อความจากผู้ใช้ผ่าน **LINE Webhook**
2. จำแนกว่าเป็นคำถามทั่วไป หรือคำถามที่ต้องดึงข้อมูลจากฐานเอกสาร (**RAG**)
3. เลือกใช้โมเดล AI ที่เหมาะสม (เช่น Google Gemini)
4. ถ้าต้องใช้ข้อมูล RAG → ดึงข้อมูลจาก **Supabase Vector Store**
5. ประมวลผลคำตอบ
6. ส่งกลับไปยังผู้ใช้ทาง LINE
7. เก็บข้อมูลทั้งหมดลงใน **Google Sheets** เพื่อเก็บ log หรือ training data

---

## 🗂️ โครงสร้างโดยรวมของ Workflow

| ลำดับ | Node | หน้าที่หลัก | การเชื่อมต่อถัดไป |
|:--:|:--|:--|:--|
| 1 | **Webhook (รับข้อความจาก LINE)** | จุดเริ่มต้น รับข้อความจากผู้ใช้ผ่าน LINE Messaging API | → Set: only message |
| 2 | **Set: only message** | ดึงเฉพาะข้อความและข้อมูลสำคัญ (เช่น replyToken) | → AI Agent classify |
| 3 | **AI Agent classify** | จำแนกประเภทข้อความ (ทั่วไป / ต้องใช้ RAG) | → classify (Code Node) |
| 4 | **classify (Code Node)** | ตรวจสอบและจัดรูปแบบผลลัพธ์การจำแนก | → Switch แยกประเภท |
| 5 | **Switch แยกประเภท** | แยกเส้นทางการทำงานตามประเภทข้อความ | → (ทั่วไป → AI Agent general)<br>(RAG → AI Agent RAG)<br>+ Google Sheets Append Log |
| 6 | **AI Agent general** | ใช้โมเดลภาษาตอบคำถามทั่วไปโดยไม่ต้องอ้างอิงฐานข้อมูล | → Clean text |
| 7 | **Clean text** | ทำความสะอาดและจัดข้อความสำหรับส่งกลับ LINE | → ส่งข้อความกลับ LINE |
| 8 | **AI Agent RAG** | ใช้โมเดลดึงข้อมูลจากฐานความรู้ Supabase เพื่อสร้างคำตอบ | → Supabase Vector Store + Embedding Ollama |
| 9 | **Supabase Vector Store + Embedding Ollama** | ค้นหาข้อมูลจากฐานเวกเตอร์ (RAG retrieval) | → Clean text1 |
| 10 | **Clean text1** | ทำความสะอาดและจัดข้อความคำตอบจาก RAG | → ส่งข้อความกลับ LINE |
| 11 | **ส่งข้อความกลับ LINE (HTTP Request)** | ส่งข้อความตอบกลับผู้ใช้ผ่าน LINE API | → สิ้นสุดการทำงาน |
| 12 | **Google Sheets Append Log** | บันทึกข้อมูล log และข้อความทั้งหมดลง Google Sheets | → สิ้นสุดการทำงาน |

---

## ⚙️ รายละเอียดแต่ละส่วน

### 🪝 1. **Webhook**
- **Node:** `Webhook`
- **หน้าที่:** จุดเริ่มต้นของ workflow รับข้อความจาก LINE Messaging API
- **Method:** `POST`
- **Path:** `/78a2555a-8272-4779-9826-5f8ea5f9cacc`
- **Output Example:**
  ```json
  {
    "body": {
      "events": [
        {
          "replyToken": "...",
          "message": { "text": "ร้านเปิดกี่โมง" },
          "source": { "userId": "Uxxxx" }
        }
      ]
    }
  }
  ```

### 🧾 2. **only message**
- **Node Type:** `Set`
- **หน้าที่:** ดึงเฉพาะข้อมูลที่ต้องใช้จาก webhook เช่นข้อความและ token

### 🤖 3. **AI Agent classify**
- ใช้โมเดล Google Gemini จำแนกข้อความว่าเป็น “RAG” หรือ “ทั่วไป”

### 🧩 4. **classify (Code Node)**
- ตรวจสอบผลลัพธ์จากโมเดล และบังคับให้เหลือเพียง `"RAG"` หรือ `"ทั่วไป"`

### 🔀 5. **Switch**
- แยกเส้นทาง workflow ออกเป็น 2 แบบ: `"RAG"` และ `"ทั่วไป"`

---

## ☕ เส้นทาง 1: คำถามทั่วไป

### 💬 6. **AI Agent general**
- ใช้โมเดลภาษาตอบคำถามทั่วไป เช่น “สวัสดีครับ วันนี้อยากดื่มกาแฟแบบไหนครับ?”

### 🧹 7. **Clean text**
- ทำความสะอาดและจัดข้อความให้อยู่ในรูป Flex Message

### 📤 8. **send LINE**
- ส่งข้อความกลับไปยัง LINE ด้วย Flex Message

---

## 📚 เส้นทาง 2: คำถามแบบ RAG

### 🧠 9. **AI Agent RAG**
- ใช้ข้อมูลจาก Supabase Vector Store เพื่อสร้างคำตอบ

### 🔍 10. **Supabase Vector Store**
- ดึงเอกสารที่คล้ายกับข้อความของผู้ใช้มากที่สุด (topK = 1)

### 📊 11. **Embeddings Ollama**
- สร้าง embedding จากข้อความเพื่อค้นหาใน Vector Store

### 🧼 12. **Clean text1**
- ทำความสะอาดผลลัพธ์ก่อนส่งกลับ LINE

### 📤 13. **send LINE RAG**
- ส่งข้อความ RAG กลับไปยังผู้ใช้ผ่าน Flex Message

---

## 🧾 14. **Google Sheets Append Log**
- บันทึกทุกข้อความลง Google Sheets เพื่อเก็บข้อมูล training และ log

---

## 🧩 Logic พิเศษใน Code Node
- ใช้ regex และ rule-based logic วิเคราะห์ intent, sentiment, keyword และเมนู

---

## 🧩 จุดเด่นของ Workflow
- รองรับ **RAG-based Chatbot** สำหรับธุรกิจจริง
- ทำงานแบบ **Hybrid AI (Rule + LLM)**
- มีระบบ Logging อัตโนมัติ
- รองรับภาษาไทยเต็มรูปแบบ
- ออกแบบให้ต่อยอดได้ เช่น เพิ่ม intent ใหม่ หรือ sentiment analysis
