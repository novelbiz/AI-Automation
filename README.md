# n8n AI Automation Tutorials

**AI Automation Series** คือชุดโปรเจกต์และวิดีโอสอนแบบลงมือทำจริง ที่ออกแบบมาเพื่อให้ทุกคนสามารถสร้างระบบอัตโนมัติ (Automation) และระบบปัญญาประดิษฐ์ (AI) ได้ด้วยตัวเอง  
โดยใช้เครื่องมือโอเพนซอร์สอย่าง **n8n**, **Gemini API**, และ **Ollama**

โปรเจกต์นี้พัฒนาโดย **บิ้ก จาก NovelBiz**  
เพื่อเปิดโลกของ “AI Workflow” ให้เห็นภาพการทำงานจริง — ว่าระบบอัตโนมัติและ AI สามารถนำไปใช้กับธุรกิจ งานคอนเทนต์ หรือระบบหลังบ้าน ได้อย่างมีประสิทธิภาพ ภายในเวลาเพียงไม่กี่นาที  

---

## 📘 สารบัญ

| EP | หัวข้อ | คำอธิบายสั้น |
|----|---------|----------------|
| 01 | [ติดตั้ง Ollama และทดสอบ API ผ่าน Postman](https://github.com/novelbiz/AI_Automation/tree/main/episodes/EP01_Ollama_Postman_API_Test) | เริ่มต้นพื้นฐานการใช้ LLM ในเครื่อง |
| 02 | [ติดตั้งและทดสอบ n8n Step by Step](https://github.com/novelbiz/AI_Automation/tree/main/episodes/EP02_n8n_Install_and_Test_StepByStep) | ปูพื้นระบบ Automation |
| 03 | [Web Scraping ลง WordPress อัตโนมัติ](https://github.com/novelbiz/AI_Automation/tree/main/episodes/EP03_n8n_WebScraping_to_WordPress) | ทำระบบ scrape + post อัตโนมัติ |
| 04 | [จากภาพสู่แคปชั่น Facebook อัตโนมัติ](https://github.com/novelbiz/AI_Automation/tree/main/episodes/EP04_Gemini_n8n_Facebook_AutoCaption) | Gemini + n8n สร้าง caption ให้อัตโนมัติ |
| 05 | [สร้างโพสต์จากวิดีโอด้วย AI](https://github.com/novelbiz/AI_Automation/tree/main/episodes/EP05_Gemini_n8n_Video_to_Post) | Gemini + n8n ทำโพสต์วิดีโอในคลิกเดียว |
| 06 | [การยืนยันตัวตน OAuth2 สำหรับ YouTube API](https://github.com/novelbiz/AI_Automation/tree/main/episodes/EP06_YouTubeAPI_OAuth2_Authen) | คู่มือสำหรับนักพัฒนา |
| 07 | [สร้างระบบ RAG ด้วย Gemini + Supabase](https://github.com/novelbiz/AI_Automation/tree/main/episodes/EP07_RAG_Gemini_Ollama_Supabase_n8n) | ดึงข้อมูลเอกสารจริงมาตอบคำถาม |
| 08 | [สร้างบอทสรุปข่าวลง Twitter](https://github.com/novelbiz/AI_Automation/tree/main/episodes/EP08_Gemini_n8n_TwitterBot_AutoNews) | n8n + Gemini อัตโนมัติ |
| 10 | [Authen Twitter API ใน n8n](https://github.com/novelbiz/AI_Automation/tree/main/episodes/EP10_TwitterAPI_Authen_in_n8n) | ใช้ API Key ทำ Automation ได้เลย |
| 11 | [สร้าง LINE Chatbot AI ฟรี](https://github.com/novelbiz/AI_Automation/tree/main/episodes/EP11_n8n_Gemini_Free_LINE_Chatbot) | ทำง่ายด้วย n8n + Gemini |
| 13 | [LINE Chatbot ฉลาดขึ้น!](https://github.com/novelbiz/AI_Automation/tree/main/episodes/EP13_n8n_LINE_Chatbot_RAG_from_Docs) | ใช้ RAG ดึงข้อมูลเอกสารจริง |
| 15 | [เชื่อม Supabase Vector Store กับ n8n](https://github.com/novelbiz/AI_Automation/tree/main/episodes/EP15_n8n_Supabase_VectorStore_RAG) | ทำ RAG Workflow ง่าย ๆ |
| 17 | [สร้าง AI Chatbot LINE ด้วย n8n + Gemini](https://github.com/novelbiz/AI_Automation/tree/main/episodes/EP17_n8n_Gemini_RAG_GoogleSheet_Chatbot) | เชื่อม Google Sheet เพื่อวิเคราะห์ข้อมูล |
| 19 | [สร้าง Product Mockups ด้วย Nano Banana](https://github.com/novelbiz/AI_Automation/tree/main/episodes/EP19_n8n_NanoBanana_Gemini_ProductMockup) | OpenRouter + Gemini 2.5 ใน n8n |



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
## 🌐 จุดเด่นของโครงการนี้

- เน้น **ภาคปฏิบัติ** ไม่ใช่แค่ทฤษฎี  
- อธิบายอย่างละเอียด เข้าใจง่าย แม้ไม่ใช่นักเขียนโค้ด  
- อัปเดตตามเทคโนโลยี AI และ Automation ล่าสุด  
- เหมาะสำหรับทั้งผู้เริ่มต้นและผู้พัฒนาในสาย AI/Automation
---

  📍**ติดตามเพิ่มเติมได้ที่:**  
- 🌐 [NovelBiz Website](https://www.novelbiz.co.th/)  
- 📺 [YouTube: NOVELBIZ](https://www.youtube.com/@NOVELBIZ/videos)  
- 💬 [Facebook: NOVELBIZ ](https://www.facebook.com/NOVELBIZThailand)
- 🎵 [TikTok: @ntechbkk](https://www.tiktok.com/@ntechbkk)

---

## 💬 สโลแกนของซีรีส์

> “ต่อยอดความคิดให้กลายเป็นระบบอัตโนมัติ — ด้วย AI ที่คุณควบคุมได้เอง”

