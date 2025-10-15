# 🐦 Twitter API: การยืนยันตัวตน (Authentication) ใน n8n ด้วย API Key

คู่มือนี้อธิบายขั้นตอนการเชื่อมต่อ **Twitter API (X)** กับ **n8n** โดยใช้ **API Key / Bearer Token / OAuth1** เพื่อทำ Automation ได้อย่างรวดเร็วและปลอดภัย เช่น การโพสต์ข้อความ, อัปโหลดรูป หรือวิดีโออัตโนมัติ

---

🎥 วิดีโอสอน:  
[![Watch the video](https://github.com/novelbiz/AI_Automation/blob/main/assets/thumbnail/Twitter_API_Auth_in_n8n.jpg)](https://youtu.be/7xsseqN_Vn8?si=zRTqZMdg36CurP-C)

---

## 🎯 วัตถุประสงค์
หลังจากทำตามคู่มือนี้ คุณจะสามารถ:
- เชื่อมต่อ Twitter API ผ่าน n8n ได้สำเร็จ  
- ใช้ API Key / Bearer Token เพื่อเรียกข้อมูลหรือโพสต์ข้อความ  
- ใช้ OAuth1 สำหรับอัปโหลดสื่อ (ภาพ / วิดีโอ)  
- สร้าง Workflow โพสต์อัตโนมัติครบวงจร

---

## 🧩 ขั้นตอนการตั้งค่า Twitter API

### 1. สมัคร Twitter Developer Account  
ไปที่ [https://developer.twitter.com](https://developer.twitter.com)  
สร้าง Developer Account และ Project ใหม่

### 2. สร้าง App และรับ API Key  
- ไปที่หน้า **Developer Portal → Projects & Apps → Your App**  
- คัดลอกข้อมูลต่อไปนี้:
  - **API Key**
  - **API Key Secret**
  - **Bearer Token**
  - **Access Token**
  - **Access Token Secret**

---

## ⚙️ ตั้งค่าใน n8n

### Node: HTTP Request (ตัวอย่างทดสอบ)
1. เพิ่ม **HTTP Request Node**  
2. ตั้งค่า:
   - **Method:** `GET`  
   - **URL:** `https://api.twitter.com/2/users/me`  
   - **Authentication:** *None*  
   - **Header:**  
     ```
     Authorization: Bearer {{$env.TWITTER_BEARER}}
     ```

3. เพิ่มในไฟล์ `.env` ของ n8n:
   ```bash
   TWITTER_BEARER=AAAAAAAAAAAAAAAAAAAAxxxYourTokenHere
   ```

4. กด **Execute Node** → หากสำเร็จจะเห็นข้อมูล user ของคุณ

---

## 💬 ตัวอย่าง Workflow โพสต์ข้อความอัตโนมัติ

**Node 1: Set Text**
```js
return [{
  json: {
    text: "สวัสดี Twitter! โพสต์นี้มาจาก n8n 🚀"
  }
}]
```

**Node 2: HTTP Request**
```
POST https://api.twitter.com/2/tweets
Header:
  Authorization: Bearer {{$env.TWITTER_BEARER}}
Body (JSON):
  {
    "text": "={{$json.text}}"
  }
```

**ผลลัพธ์:** ทวีตถูกโพสต์ขึ้นบนบัญชีของคุณโดยอัตโนมัติ 🎉

---

## 🖼️ #Node Upload Media

การอัปโหลดภาพหรือวิดีโอต้องใช้ **Twitter Upload API (v1.1)**  
เพราะ endpoint ของ v2 ยังไม่รองรับการ upload media โดยตรง

**Endpoint:**  
```
POST https://upload.twitter.com/1.1/media/upload.json
```

**Authentication:** OAuth1 (ใช้ Access Token + Secret)  

### ⚙️ ตั้งค่าใน n8n

1. เพิ่ม **HTTP Request Node**  
2. ตั้งค่า:
   - **Method:** `POST`  
   - **URL:** `https://upload.twitter.com/1.1/media/upload.json`
   - **Authentication:** OAuth1 API  
   - **OAuth1 Credentials:**
     ```
     Consumer Key (API Key)
     Consumer Secret (API Key Secret)
     Access Token
     Access Token Secret
     ```
3. **Body (Form Data):**
   ```
   media=@/path/to/your/image.jpg
   ```
4. เมื่ออัปโหลดสำเร็จจะได้ค่ากลับมา เช่น:
   ```json
   {
     "media_id": "1923847293847",
     "media_id_string": "1923847293847"
   }
   ```
5. ใช้ `media_id_string` นี้แนบไปกับการโพสต์ข้อความ

---

## 🧠 โพสต์ข้อความพร้อมภาพ/วิดีโอ

**Node: HTTP Request**
```
POST https://api.twitter.com/2/tweets
Header:
  Authorization: Bearer {{$env.TWITTER_BEARER}}
Body (JSON):
{
  "text": "โพสต์นี้มาพร้อมภาพจาก n8n 🎬",
  "media": {
    "media_ids": ["={{$json.media_id_string}}"]
  }
}
```

---

## 🔐 OAuth1 API ข้อมูลอ้างอิง

| รายการ | URL |
|--------|-----|
| Authorization URL | https://api.twitter.com/oauth/authorize |
| Access Token URL | https://api.twitter.com/oauth/access_token |
| Request Token URL | https://api.twitter.com/oauth/request_token |

> ใช้ข้อมูลนี้เมื่อสร้าง OAuth1 Credentials ใน n8n เพื่อใช้กับ Node Upload Media

---

## 💡 Tips & Best Practices

- ใช้ **Environment Variables** แทนการใส่ Token ตรง ๆ ใน Node  
- อัปโหลดไฟล์ก่อนโพสต์เสมอ ถ้ามีสื่อแนบ  
- รองรับเฉพาะไฟล์ `.jpg`, `.png`, `.gif`, `.mp4` (ตามประเภทบัญชี)  
- หากต้องโพสต์วิดีโอขนาดใหญ่ ให้แบ่งเป็น `INIT → APPEND → FINALIZE` ตามขั้นตอน Media Upload API  
- ใช้ Error Workflow ใน n8n เพื่อ handle “Rate Limit” หรือ “Invalid Token”

---

## 📚 เอกสารอ้างอิง

- [Twitter API v2 Docs](https://developer.twitter.com/en/docs/twitter-api)
- [Media Upload API v1.1](https://developer.twitter.com/en/docs/twitter-api/v1/media/upload-media/overview)
- [n8n HTTP Request Docs](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.httprequest/)

---

> จัดทำโดย: **NOVELBIZ Automation Lab**  
> อัปเดตล่าสุด: 15 ตุลาคม 2025  
