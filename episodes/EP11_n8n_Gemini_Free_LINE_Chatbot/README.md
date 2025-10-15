# 🤖 LINE Chatbot ด้วย n8n + Google Gemini

Workflow นี้สร้าง **AI Chatbot บน LINE** โดยใช้ **n8n** เชื่อมต่อกับ **Google Gemini** (ผ่าน LangChain Agent)  
เพื่อให้บอทสามารถตอบกลับข้อความจากผู้ใช้ได้อย่างเป็นธรรมชาติและสุภาพ 💬✨

---

🎥 วิดีโอสอน:  
[![Watch the video](https://github.com/novelbiz/AI_Automation/blob/main/assets/thumbnail/line%20chat%20bot.png)](https://youtu.be/0N3X9lG6n2c?si=bxz-EYSnoLIo2lJL)

---

## 🧩 โครงสร้างโดยรวมของ Workflow

### 🔗 1. Webhook (Trigger)
- **Node:** `Webhook`
- **Type:** `n8n-nodes-base.webhook`
- **Method:** `POST`
- **Path:** `8ca874e4-80b3-403d-875a-7065ebfda19f`
- **หน้าที่:**  
  รับข้อมูลจาก LINE Messaging API เมื่อผู้ใช้ส่งข้อความเข้ามา

---

### 🧱 2. Edit Fields (Set)
- **Node:** `Edit Fields`
- **Type:** `n8n-nodes-base.set`
- **ทำหน้าที่:**  
  ดึงค่าที่สำคัญจากข้อมูลของ LINE ได้แก่  
  - `body.events[0].message.text` → ข้อความที่ผู้ใช้ส่งมา  
  - `body.events[0].replyToken` → token ที่ใช้ตอบกลับผู้ใช้  

```js
{
  "body.events[0].message.text": "={{ $json.body.events[0].message.text }}",
  "body.events[0].replyToken": "={{ $json.body.events[0].replyToken }}"
}
```

---

### 🧠 3. AI Agent (LangChain Agent)
- **Node:** `AI Agent`
- **Type:** `@n8n/n8n-nodes-langchain.agent`
- **หน้าที่:**  
  ใช้ **Google Gemini** เป็นโมเดลภาษาหลัก (Language Model) เพื่อสร้างข้อความตอบกลับ  
  พร้อมระบบข้อความ (System Message) ที่กำหนดแนวทางการพูด เช่น  
  - สุภาพ เป็นกันเอง  
  - ใช้ภาษาธรรมชาติ  
  - สามารถใส่อีโมจิได้  
  - หลีกเลี่ยงอักขระพิเศษ เช่น `\n`, `\r`, `\`  

**Prompt ที่ส่งให้โมเดล:**
```text
นี่คือข้อความที่ผู้ใช้ส่งมาใน LINE: "{{ $json.body.events[0].message.text }}"
โปรดตอบกลับด้วยภาษาที่สุภาพ เป็นกันเอง และอ่านง่าย
ใช้ภาษาธรรมชาติ และสามารถใส่อีโมจิได้ถ้าเหมาะสม
ตอบให้ตรงประเด็น เข้าใจง่าย และเป็นมิตร
```

---

### 🌐 4. Google Gemini Chat Model
- **Node:** `Google Gemini Chat Model`
- **Type:** `@n8n/n8n-nodes-langchain.lmChatGoogleGemini`
- **Credentials:** `Google Gemini(PaLM) Api account`
- **หน้าที่:**  
  เป็นโมเดลภาษา (LLM) ที่เชื่อมต่อกับ Agent เพื่อให้ AI สามารถตอบกลับได้อย่างชาญฉลาด

---

### 🧮 5. Code (Clean JSON)
- **Node:** `Code`
- **Type:** `n8n-nodes-base.code`
- **หน้าที่:**  
  ทำความสะอาดข้อมูลที่ได้จาก AI ก่อนจะส่งกลับไปยัง LINE  
  โดยลบอักขระ `\n`, `\r`, `/` และดึงเฉพาะส่วนที่เป็น JSON ที่ถูกต้อง

**ตัวอย่างโค้ดที่ใช้:**
```js
let rawData = $input.first().json.output ?? $input.first().json;

if (typeof rawData !== 'string') {
  rawData = JSON.stringify(rawData);
}

rawData = rawData.replace(/\r?\n|\r/g, '').replace(/\//g, '');

const match = rawData.match(/\{[\s\S]*\}/);

if (match) {
  try {
    const cleanJson = JSON.parse(match[0]);
    return [{ json: cleanJson }];
  } catch (err) {
    return [{ json: { error: 'Invalid JSON after cleaning', raw: match[0] } }];
  }
} else {
  return [{ json: { error: 'No JSON found in input', raw: rawData } }];
}
```

---

### 💾 6. Simple Memory
- **Node:** `Simple Memory`
- **Type:** `@n8n/n8n-nodes-langchain.memoryBufferWindow`
- **หน้าที่:**  
  เก็บประวัติการสนทนาแบบง่าย (Memory Buffer)  
  โดยสร้าง Session Key แบบสุ่มสำหรับแต่ละผู้ใช้

---

### 💬 7. Reply to LINE (HTTP Request)
- **Node:** `reply to line`
- **Type:** `n8n-nodes-base.httpRequest`
- **Method:** `POST`
- **URL:** `https://api.line.me/v2/bot/message/reply`
- **Headers:**
  ```json
  {
    "Content-Type": "application/json",
    "Authorization": "Bearer [LINE_CHANNEL_ACCESS_TOKEN]"
  }
  ```
- **Body:**
  ```json
  {
    "replyToken": "{{ $('Edit Fields').item.json.body.events[0].replyToken }}",
    "messages": [
      {
        "type": "text",
        "text": "{{ $json.raw }}"
      }
    ]
  }
  ```

---

## 🔄 การเชื่อมต่อระหว่าง Node

| จาก | ไปยัง | รายละเอียด |
|-----|--------|-------------|
| Webhook | Edit Fields | รับข้อความจาก LINE |
| Edit Fields | AI Agent | ส่งข้อความผู้ใช้ไปให้ AI ประมวลผล |
| Google Gemini Chat Model | AI Agent | ใช้โมเดล Gemini เป็น LLM |
| Simple Memory | AI Agent | จัดการ memory การสนทนา |
| AI Agent | Code | ส่งผลลัพธ์ข้อความให้ทำความสะอาด |
| Code | reply to line | ส่งข้อความตอบกลับกลับไปที่ LINE |

---

## 🚀 วิธีใช้งาน

1. สร้าง Workflow ในน8n แล้วนำ JSON ด้านบนไป Import  
2. ตั้งค่า Credential สำหรับ
   - **Google Gemini (PaLM API)**
   - **LINE Channel Access Token**
3. Deploy Workflow ให้พร้อมใช้งาน  
4. ตั้งค่า Webhook URL ไปที่ **LINE Developer Console** (Messaging API → Webhook URL)  
5. ทดสอบส่งข้อความจาก LINE แล้วรอดูคำตอบจาก AI 🤖💬

---

## 🧠 สรุป
Workflow นี้จะช่วยให้คุณสร้าง **LINE Chatbot** ที่ตอบกลับอัตโนมัติด้วย **AI Google Gemini**  
โดยใช้ **n8n** เป็น Workflow Automation ไม่ต้องเขียนโค้ดมาก 🔧  
สามารถปรับแต่งข้อความตอบกลับ สไตล์การพูด หรือเพิ่ม Node อื่น ๆ เช่น Google Sheet, Database, API ต่าง ๆ ได้อย่างอิสระ 🎯  

---

> ✨ **พัฒนาโดย:** ระบบอัตโนมัติ n8n  
> 🤝 **ผสานพลัง:** LINE Messaging API + Google Gemini (LangChain)
