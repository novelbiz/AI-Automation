# 🎨 สร้าง Product Mockups ด้วย Nano Banana (OpenRouter + Gemini 2.5) ใน n8n

> 💡 โฟลว์นี้เป็น `n8n` workflow สำหรับรับภาพสามไฟล์ (เสื้อ 👕, รองเท้า 👟, โมเดล 🧍‍♀️) จากฟอร์ม → แปลงไฟล์ → สร้าง prompt → ส่งไปยังโมเดลภาพ (Gemini / OpenRouter) → ดึง URL ภาพผลลัพธ์📸  

---

🎥 วิดีโอสอน: 
[![Watch the video](https://github.com/novelbiz/AI_Automation/blob/main/assets/thumbnail/Generate%20Product%20Mockups%20with%20Nano%20Banana.png)](https://youtu.be/TC5sTjpcEws?si=LT4NroL6Iwbob2nF)

---

## 🧩 ภาพรวม (High-level)

| ลำดับ | ขั้นตอน                                          | รายละเอียด                                                                      |
| :---: | :----------------------------------------------- | :------------------------------------------------------------------------------ |
|  1️⃣  | **รับข้อมูลจากฟอร์ม**                            | ผู้ใช้ส่งฟอร์มที่มีไฟล์ 3 รูป: `shirt`, `shoe`, `model`                         |
|  2️⃣  | **แยกไฟล์ภาพออกเป็น property**                   | ใช้ `extractFromFile` เพื่อให้เข้าถึงข้อมูลภาพได้ใน `$json.data`                |
|  3️⃣  | **ส่งข้อมูลไปยังโมเดลภาพ (Gemini / OpenRouter)** | ส่งภาพแบบ inline base64 เพื่อให้โมเดลสร้าง prompt / วิเคราะห์ / สร้างภาพ mockup |
|  4️⃣  | **ทำความสะอาดข้อความ prompt**                    | ลบอักขระพิเศษ / newline และจัดรูปแบบข้อความให้สะอาดก่อนใช้                      |
|  5️⃣  | **เรียกใช้งาน OpenRouter API**                   | ส่ง prompt + ภาพ เพื่อให้โมเดลสร้างภาพตามที่กำหนด                               |
|  6️⃣  | **จัดการผลลัพธ์ภาพที่ได้**                       | ดึง URL หรือ base64 ของภาพ → แปลงเป็นไฟล์สำหรับขั้นตอนต่อไป                     |

---

## ⚙️ Prerequisites / การตั้งค่าก่อนรัน

* 🧩 ติดตั้งและรัน `n8n` ได้
* 🔐 ตั้งค่า credentials / secrets:
  * `GEMINI_API_KEY_PLACEHOLDER`
  * `OPENROUTER_CREDENTIAL_ID_PLACEHOLDER`
  * `FACEBOOK_ACCESS_TOKEN_PLACEHOLDER`
* 🌐 ตรวจสอบว่า webhook ของ `On form submission` เปิดใช้งานได้
* 💾 n8n ต้องอ่าน/เขียน binary data ได้
* 📏 ตรวจสอบขนาดไฟล์ upload ให้เหมาะสม

---

## 🧱 โครงสร้าง Node-by-Node

### 1️⃣ `On form submission` (🧾 formTrigger)
รับข้อมูลจากฟอร์มชื่อ **Nano Banana**  
📤 Output: binary data ของไฟล์ทั้ง 3

---

### 2️⃣ `Upload__shirt` 👕 (extractFromFile)
แปลง binary ของเสื้อให้เป็น base64 ที่เข้าถึงได้ใน `$json.data`

---

### 3️⃣ `Upload__shoe` 👟
เหมือนกับเสื้อ แต่เป็นรองเท้า

---

### 4️⃣ `Extract from File` 🧍‍♀️ (โมเดล)
แปลง binary ของภาพโมเดลให้เป็น property base64

---

### 5️⃣ `gemini image to Fabric type` 🤖 (HTTP → Gemini)
* วิเคราะห์ภาพทั้งสามเพื่อสร้าง **prompt สำหรับสร้างภาพ mockup**
* ใช้ `inline_data` ของรูปทั้งหมด (เสื้อ/รองเท้า/โมเดล)
* 📡 API: `https://generativelanguage.googleapis.com/v1beta/...`

---

### 6️⃣ `Code` 💻
* ล้างข้อความ prompt — ลบอักขระพิเศษ / newline
* คืนค่า `{ json: { prompt: cleanPrompt } }`

### ตัวอย่างโค้ด Node (Clean Prompt)

```js

// รับข้อความจาก input
const prompt = $input.first().json.candidates[0].content.parts[0].text || "";

// ลบอักขระพิเศษ และ \n ให้เหลือแต่ ตัวอักษร ตัวเลข และช่องว่าง
const cleanPrompt = prompt.replace(/[^a-zA-Z0-9ก-๙\s]/g, " ").replace(/\s+/g, " ").trim();

return [
  {
    json: {
      prompt: cleanPrompt
    }
  }
];
```

---

### 7️⃣ `Assemble Final Data` 🧩
รวมข้อมูลทั้งหมด (prompt + base64 ของภาพทั้งสาม)  
> ⚠️ ระวัง property `" Upload__shirt"` ที่มีช่องว่างนำหน้า

---

### 8️⃣ `openrouter` 🌐 (HTTP → OpenRouter)
* ส่ง prompt + ภาพ (base64 data-uri) เพื่อให้โมเดล **generate image**
* `model`: `google/gemini-2.5-flash-image-preview:generateContent`

---

### 9️⃣ `If` 🔍
ตรวจสอบว่าผลลัพธ์จาก `openrouter` มี `image_url` หรือไม่  
* ✅ ถ้ามี → ดำเนินต่อ  
* ❌ ถ้าไม่มี → กลับไป `Upload__shirt` (retry)

---

### 🔟 `Edit Fields` ✏️ / `Edit Fields1` ✏️
* ดึงค่า base64 จาก URL (`split(",")[1]`)
* เตรียมข้อมูลภาพสำหรับขั้นตอนแปลงเป็น binary

---

### 11️⃣ `Convert to File` 🗂️
แปลง base64 → binary file เพื่อส่งให้ API ถัดไป

---

## ⚠️ จุดที่ควรระวัง (Pitfalls)

1. 🧱 **ชื่อ property มีช่องว่าง** (`" Upload__shirt"`)
2. 🔄 **Assumption รูปแบบ response** (`split(",")[1]`)
3. 💾 **ขนาดไฟล์ใหญ่** → เสี่ยง memory overflow
4. 🧯 **ไม่มี retry/backoff หรือ error log**
5. 🕵️‍♂️ **Secrets exposure**
6. 📊 **Rate limits / Quotas**

---


## 🧰 ตัวอย่าง JSON Body

```json
{
  "contents": [
    {
      "parts": [
        { "text": "ช่วยเขียน prompt สร้างภาพ ..." },
        { "inline_data": { "mime_type": "image/jpeg", "data": "{{ $('Upload__shirt').item.json.data }}" } },
        { "inline_data": { "mime_type": "image/jpeg", "data": "{{ $('Upload__shoe').item.json.data }}" } },
        { "inline_data": { "mime_type": "image/jpeg", "data": "{{ $json.data }}" } }
      ]
    }
  ]
}
```

---

## 🧩 Troubleshooting เบื้องต้น

| ปัญหา | วิธีตรวจสอบ |
|--------|---------------|
| ❌ ไม่เห็นผลจาก OpenRouter | ดู log ของ node `openrouter` |
| 🧃 ภาพแตก / ไม่ครบ | ตรวจสอบ base64 และ `mime_type` |

---

## 🧠 ข้อแนะนำเพิ่มเติม

* 🧪 ทดสอบด้วยภาพขนาดเล็กก่อนใช้งานจริง  
* 🔧 ใช้ manual trigger ก่อนเปิด active  
* 💾 เก็บ prompt template ที่ดีไว้ใช้ซ้ำ  
* 🧰 บันทึกเวอร์ชันของ node และ credential config

---

## ✅ สรุป (Final Notes)

* โฟลว์: **รับไฟล์ → แปลง → ส่งให้โมเดล → ทำความสะอาด prompt → เรียก OpenRouter → รับ image **
* จุดสำคัญ: **ชื่อ property**, **response format**, **การจัดการไฟล์ base64/URL**
* 🔧 แนะนำ: แก้ชื่อ property, ใช้ storage แทน inline, และเพิ่ม logging/retry mechanism





