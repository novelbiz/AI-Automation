# n8n AI Automation Tutorials

**AI Automation Series** คือชุดโปรเจกต์และวิดีโอสอนเชิงปฏิบัติ ที่ออกแบบมาเพื่อให้ทุกคนสามารถสร้างระบบอัตโนมัติ (Automation) และระบบปัญญาประดิษฐ์ (AI) ได้ด้วยตัวเอง โดยใช้เครื่องมือโอเพนซอร์สอย่าง **n8n**, **Gemini API**, และ **Ollama**

โปรเจกต์นี้พัฒนาโดย **บิ้ก จาก NovelBiz** เพื่อเปิดโลกของ “AI Workflow” ให้ทุกคนได้เห็นภาพจริง ว่าสามารถนำไปใช้กับธุรกิจ งานคอนเทนต์ และระบบหลังบ้านได้จริงภายในไม่กี่นาที

---

## 🎯 เป้าหมายของโครงการ

- สอนให้เข้าใจการสร้าง **AI Workflow** แบบ Step-by-Step  
- เชื่อมต่อ **API ของแพลตฟอร์มยอดนิยม** เช่น Facebook, YouTube, LINE, Twitter (X)  
- เรียนรู้การใช้ **Gemini API**, **Ollama (Local LLM)** และ **Supabase Vector Store** เพื่อสร้างระบบ RAG (Retrieval-Augmented Generation)  
- สร้างระบบที่ทำงานได้จริง เช่น  
  - ดึงข่าวอัตโนมัติแล้วโพสต์ลง X  
  - แปลงภาพเป็นแคปชันอัตโนมัติ  
  - ทำ LINE Chatbot ที่ตอบจากเอกสารจริง  
  - เชื่อม AI กับ Google Sheet เพื่อวิเคราะห์ข้อมูลธุรกิจ

---

## ⚙️ เทคโนโลยีที่ใช้

- **n8n** – ระบบ Automation แบบ Visual Workflow  
- **Google Gemini API** – โมเดล AI สำหรับการประมวลผลข้อความและภาพ  
- **Ollama** – เครื่องมือรัน LLM ในเครื่อง เช่น Llama 3, Phi 3  
- **Supabase Vector Store** – สำหรับเก็บและค้นหาข้อมูลฝัง (Embeddings)  
- **Postman / cURL** – ใช้ทดสอบ API ก่อนเชื่อมจริง  
- **Twitter API / LINE Messaging API / YouTube API** – สำหรับการเชื่อมต่อแพลตฟอร์มโซเชียล

---

## 🧩 โครงสร้างซีรีส์

ซีรีส์แบ่งเป็นตอน (EP) ตั้งแต่พื้นฐานจนถึงระดับโปร แต่ละตอนจะมีโฟลเดอร์แยก เช่น:

```
EP01_Ollama_Postman_API_Test/
EP02_n8n_Install_and_Test_StepByStep/
EP07_RAG_Gemini_Ollama_Supabase_n8n/
EP17_n8n_Gemini_RAG_GoogleSheet_Chatbot/
```

ภายในจะมี `workflow.json`, `setup_instructions.md`, `screenshots/` และตัวอย่างโค้ดครบทุกตอน

---

## 🌐 จุดเด่นของโครงการนี้

- เน้น **ภาคปฏิบัติ** ไม่ใช่แค่ทฤษฎี  
- อธิบายอย่างละเอียด เข้าใจง่าย แม้ไม่ใช่นักเขียนโค้ด  
- อัปเดตตามเทคโนโลยี AI และ Automation ล่าสุด  
- เหมาะสำหรับทั้งผู้เริ่มต้นและผู้พัฒนาในสาย AI/Automation

---

## 📘 รายการตอน

| EP | หัวข้อ | คำอธิบายสั้น |
|----|---------|----------------|
| 01 | ติดตั้ง Ollama และทดสอบ API ผ่าน Postman | เริ่มต้นพื้นฐานการใช้ LLM ในเครื่อง |
| 02 | ติดตั้งและทดสอบ n8n Step by Step | ปูพื้นระบบ Automation |
| 03 | Web Scraping ลง WordPress อัตโนมัติ | ทำระบบ scrape + post อัตโนมัติ |
| 04 | จากภาพสู่แคปชั่น Facebook อัตโนมัติ | Gemini + n8n สร้าง caption ให้อัตโนมัติ |
| 05 | สร้างโพสต์จากวิดีโอด้วย AI | Gemini + n8n ทำโพสต์วิดีโอในคลิกเดียว |
| 06 | การยืนยันตัวตน OAuth2 สำหรับ YouTube API | คู่มือสำหรับนักพัฒนา |
| 07 | สร้างระบบ RAG ด้วย Gemini + Supabase | ดึงข้อมูลเอกสารจริงมาตอบคำถาม |
| 08 | สร้างบอทสรุปข่าวลง Twitter | n8n + Gemini อัตโนมัติ |
| 09 | แนะนำ บริษัท โนเวลบิซ จำกัด | รู้จักทีมและวิธีทำงาน |
| 10 | Authen Twitter API ใน n8n | ใช้ API Key ทำ Automation ได้เลย |
| 11 | สร้าง LINE Chatbot AI ฟรี | ทำง่ายด้วย n8n + Gemini |
| 13 | LINE Chatbot ฉลาดขึ้น! | ใช้ RAG ดึงข้อมูลเอกสารจริง |
| 15 | เชื่อม Supabase Vector Store กับ n8n | ทำ RAG Workflow ง่าย ๆ |
| 17 | สร้าง AI Chatbot LINE ด้วย n8n + Gemini | เชื่อม Google Sheet เพื่อวิเคราะห์ข้อมูล |
| 19 | สร้าง Product Mockups ด้วย Nano Banana | OpenRouter + Gemini 2.5 ใน n8n |

---

## 💬 สโลแกนของซีรีส์

> “ต่อยอดความคิดให้กลายเป็นระบบอัตโนมัติ — ด้วย AI ที่คุณควบคุมได้เอง”

