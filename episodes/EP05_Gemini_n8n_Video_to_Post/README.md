# 🤖 AI Video Content Creator Workflow (n8n & Gemini)

นี่คือ **n8n Workflow** ที่ออกแบบมาเพื่อดึงวิดีโอล่าสุดจาก **Google Drive**, ใช้ **AI (Gemini)** เพื่อสร้าง **แคปชั่นและ Metadata** (Title, Description, Tags) จากภาพตัวอย่าง (Thumbnail) ของวิดีโอ จากนั้นอัปโหลดวิดีโอพร้อมข้อมูลที่สร้างโดย AI ไปยัง **YouTube** และ **Facebook** โดยอัตโนมัติ
---

🎥 วิดีโอสอน: 
[![Watch the video](https://github.com/novelbiz/AI_Automation/blob/main/episodes/EP05_Gemini_n8n_Video_to_Post/Image/upvideo%20to%20facebook%20youtube.png)](https://youtu.be/hlx_YP-Auso?si=fX1SUNbJqlvbD-EH)

---

## ⚙️ Workflow Overview

1. **ค้นหาวิดีโอ:** ดึงวิดีโอล่าสุดจากโฟลเดอร์ที่กำหนดใน Google Drive
2. **ดาวน์โหลดและเตรียมข้อมูล:** ดาวน์โหลดวิดีโอและกำหนดชื่อไฟล์ชั่วคราวสำหรับ Frame/Thumbnail
3. **สร้าง Thumbnail:** ใช้ FFmpeg เพื่อดึงภาพ Frame ที่ 10 ของวิดีโอเป็นภาพตัวอย่าง
4. **สร้างแคปชั่นด้วย Gemini:** ส่งภาพ Thumbnail ไปยัง Gemini เพื่อสร้างหัวข้อโพสต์สั้น ๆ
5. **สร้าง Metadata YouTube/Facebook:** ใช้ AI Agent เพื่อขยายหัวข้อสั้น ๆ เป็น Title, Description และ Tags
6. **อัปโหลดวิดีโอ:** ส่งวิดีโอพร้อม Metadata และ Thumbnail ไปยัง YouTube และ Facebook

---

## 🛠️ Prerequisites

* **n8n** ติดตั้งและพร้อมใช้งาน
* **FFmpeg** ติดตั้งและตั้งค่า PATH ให้เรียกใช้จาก command line ได้
* **Google Drive API Key** สำหรับการดึงไฟล์
* **YouTube API Key / OAuth Credential** สำหรับอัปโหลดวิดีโอ
* **Facebook Graph API Access Token** สำหรับอัปโหลดวิดีโอ

> 💡 Tips: สร้าง `.env` หรือ Environment Variables สำหรับ API Key เพื่อให้ workflow ปลอดภัยและง่ายต่อการปรับเปลี่ยน

---

## 🗃️ Nodes Configuration & Key Parameters

### 1️⃣ Google Drive (Search & Download)

* **Purpose:** ค้นหาวิดีโอใหม่จากโฟลเดอร์ที่กำหนด
* **QueryString:** `mimeType contains 'video/'`
* **Limit:** `1`
* **Credential:** Google Drive OAuth2 / Service Account

---

### 2️⃣ Code Node (Generate Temporary Paths)

```javascript
return [
  {
    json: {
      fileName: $json["fileName"],
      frameImageName: (() => {
        const now = new Date();
        const datePart = now.toISOString().slice(0, 10);
        return `D:/tmp/frame_${datePart}.png`;
      })(),
    },
  },
];
```

---

### 3️⃣ Extract Frame (FFmpeg)

```bash
ffmpeg -y -i "{{ $json.fileName }}" -vf "select=eq(n\,10)" -vframes 1 "{{ $json.frameImageName }}"
```

---

### 4️⃣ Gemini Image to Caption

* **URL:** `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-exp-image-generation:generateContent?`
* **JSON Body:**

```json
{
  "contents": [
    {
      "parts": [
        {"text": "ช่วยเขียนหัวข้อ สำหรับโพสต์บน Facebook และ Youtube โดยขอแค่ 1 คำตอบเท่านั้น"},
        {"inline_data": {"mime_type": "image/jpeg", "data": "{{ $json.data }}"}}
      ]
    }
  ]
}
```

---

### 5️⃣ AI Agent

```markdown
Create a YouTube video title, description, and tags for a video about '{{ $json.candidates[0].content.parts[0].text }}'. Format the output in JSON with fields: title, description, and tags (as an array). All tags must start with a '#' symbol. Use Thai language. Friendly and motivational tone.
```

---

### 6️⃣ Structured Output Parser

```json
{
  "snippet": {
    "title": "",
    "description": "",
    "tags": [""],
    "categoryId": "22"
  },
  "status": {"privacyStatus": "unlisted"}
}
```

---

### 7️⃣ Upload Content to YouTube (Metadata)

```json
{
  "snippet": {
    "title": "{{ $json.output.snippet.title }}",
    "description": "{{ $json.output.snippet.description.replace(/\r?\n/g, '') }}",
    "categoryId": "22"
  },
  "status": {"privacyStatus": "unlisted"}
}
```

> Node นี้จะได้ค่า `location` header สำหรับใช้ในขั้นตอนอัปโหลดไฟล์จริง

---

### 8️⃣ Upload Video to YouTube (Actual File)

* **Method:** `PUT`
* **URL:** `={{ $json.headers.location }}`

---

### 9️⃣ Facebook Graph API

* **Host URL:** `graph-video.facebook.com`
* **Edge:** `videos`
* **Query Parameters:**

  * `description`: `={{ $('AI Agent').item.json.output.snippet.description }}`
  * `published`: `true`

---

### 🔟 Upload Image Thumbnails to YouTube

* **URL:** `[https://www.googleapis.com/upload/youtube/v3/thumbnails/set?videoId=](https://www.googleapis.com/upload/youtube/v3/thumbnails/set?videoId=){{ $('upload-video-to-you-tube1').item.json.id }}`

---

## 💡 Tips & Best Practices

* ตรวจสอบว่า API Key และ Token ถูกต้องก่อนรัน workflow
* สำหรับวิดีโอใหญ่ แนะนำตั้งค่า Chunked Upload หรือ Polling Interval
* เก็บไฟล์ชั่วคราวใน tmp/ และลบไฟล์เก่าเป็นประจำ
* ใช้ Environment Variables สำหรับ Key และ Credential เพื่อความปลอดภัย

---

## 🚀 How to Run

1. Import workflow `ai_video_workflow.json` ใน n8n
2. ตั้งค่า Credential สำหรับ Google Drive, YouTube, Facebook
3. ตรวจสอบ FFmpeg สามารถเรียกใช้จาก command line ได้
4. Run workflow
5. ตรวจสอบผลลัพธ์บน YouTube และ Facebook

> พร้อม! workflow ของคุณจะทำงานแบบอัตโนมัติและโพสต์วิดีโอพร้อม Metadata และ Thumbnail โดยใช้ AI
