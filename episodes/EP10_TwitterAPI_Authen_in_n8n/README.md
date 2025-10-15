# 🐦 Twitter API: การยืนยันตัวตน (Authentication) ใน n8n ด้วย API Key

คู่มือนี้อธิบายขั้นตอนการเชื่อมต่อ **Twitter API (X)** กับ **n8n** โดยใช้ **API Key & Bearer Token** เพื่อทำ Automation ได้อย่างรวดเร็วและปลอดภัย

---

🎥 วิดีโอสอน: 
[![Watch the video](https://github.com/novelbiz/AI_Automation/blob/main/assets/thumbnail/Twitter_API_Auth_in_n8n.jpg)](https://youtu.be/7xsseqN_Vn8?si=zRTqZMdg36CurP-C)

---

## 🎯 วัตถุประสงค์
หลังจากทำตามคู่มือนี้ คุณจะสามารถ:
- เชื่อมต่อ Twitter API ผ่าน n8n ได้สำเร็จ
- ใช้ API Key / Bearer Token เพื่อเรียกข้อมูลหรือโพสต์อัตโนมัติ
- สร้าง Workflow ที่โพสต์ข้อความไปยัง Twitter ได้ทันที

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
  - (หากต้องการเขียนโพสต์: สร้าง Access Token และ Secret ด้วย)

### 3. ทดสอบด้วย Postman (แนะนำ)
ตัวอย่างทดสอบ API:  
```
GET https://api.twitter.com/2/users/me
Header:
Authorization: Bearer <YOUR_BEARER_TOKEN>
```

---

## ⚙️ ตั้งค่าใน n8n

### Node: HTTP Request
1. เพิ่ม **HTTP Request Node**
2. ตั้งค่า:
   - **Method:** `GET`
   - **URL:** `https://api.twitter.com/2/users/me`
   - **Authentication:** *None*
   - **Header:**  
     `Authorization: Bearer {{$env.TWITTER_BEARER}}`

3. ไปที่ **.env ของ n8n** เพิ่มบรรทัดนี้:
   ```bash
   TWITTER_BEARER=AAAAAAAAAAAAAAAAAAAAxxxYourTokenHere
   ```

4. กด Execute Node → หากเชื่อมต่อสำเร็จจะเห็นข้อมูล user ของคุณ

---

## 🧠 ตัวอย่าง Workflow อัตโนมัติ

### โพสต์ข้อความลง Twitter อัตโนมัติ

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

## 💡 Tips & Best Practices

- อย่าระบุ API Key หรือ Token ใน Node โดยตรง — ใช้ Environment Variables แทน
- หากต้องการโพสต์สื่อ (ภาพ/วิดีโอ) ต้องใช้ endpoint `media/upload` ก่อน
- ตั้งค่า Workflow ให้มี Node ตรวจสอบ Error เช่น “rate limit” หรือ “invalid token”
- ใช้ Combine Node หรือ Split In Batches เพื่อโพสต์หลายข้อความแบบ queue

---

## 📚 เอกสารอ้างอิง
- [Twitter API v2 Docs](https://developer.twitter.com/en/docs/twitter-api)
- [n8n HTTP Request Docs](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.httprequest/)

---

## 🎬 แนะนำวิดีโอสอน (YouTube)
[สอน Authen Twitter API ใน n8n ใช้ API KEY ทำ Automation ได้เลย!](https://youtu.be/7xsseqN_Vn8?si=2R3oCIR4la3m71d0)

---

> จัดทำโดย: **NOVELBIZ Automation Lab**  
> อัปเดตล่าสุด: 15 ตุลาคม 2025
