# 🤖 Automate Caption Gemini: สร้างแคปชั่น Facebook อัตโนมัติจากรูปภาพด้วย n8n และ Gemini API

เวิร์กโฟลว์ **Automate Caption Gemini** เป็นโซลูชันสำหรับการทำการตลาดบนโซเชียลมีเดียที่ขับเคลื่อนด้วย AI (AI-Powered Social Media Marketing) โดยใช้ **n8n** เพื่อดึงรูปภาพจาก Google Drive และให้โมเดล **Gemini (ผ่าน Google Gen AI API)** วิเคราะห์รูปภาพและสร้างแคปชั่นพร้อมแฮชแท็กที่น่าสนใจ จากนั้นโพสต์ลง Facebook Page โดยอัตโนมัติตามกำหนดเวลา

ช่วยให้คุณประหยัดเวลาในการคิดคอนเทนต์ และสร้างโพสต์ที่น่าดึงดูดได้อย่างสม่ำเสมอ

---

## ✨ คุณสมบัติหลัก

* **ตั้งเวลาทำงานอัตโนมัติ (Schedule Trigger):** กำหนดให้เวิร์กโฟลว์ทำงานทุกวันในเวลาที่คุณต้องการ (ตั้งค่าไว้ที่ 9 โมงเช้า).
* **ค้นหารูปภาพล่าสุด:** ดึงรูปภาพใหม่ล่าสุดที่ถูกอัปโหลดไปยังโฟลเดอร์ Google Drive ที่กำหนดในแต่ละวัน.
* **AI สร้างแคปชั่น:** ใช้ **Gemini API** เพื่อวิเคราะห์รูปภาพสินค้าแฟชั่น และสร้างแคปชั่นขายของที่ "เท่ๆ" พร้อมแฮชแท็กโดยอัตโนมัติ.
* **โพสต์ Facebook อัตโนมัติ:** โพสต์รูปภาพพร้อมแคปชั่นที่สร้างโดย AI ไปยัง Facebook Page ของคุณ.

---

## 🛠️ การตั้งค่าที่จำเป็น (Prerequisites)

ก่อนใช้งาน คุณต้องมีบัญชีและข้อมูลสำคัญเหล่านี้:

1.  **n8n Instance:** (Self-hosted หรือ Cloud)
2.  **Google Drive Account**
3.  **Facebook Page** และ **Facebook Access Token** (ต้องมีสิทธิ์โพสต์ในเพจ).
4.  **Google Generative AI (Gemini) API Key**

---

## ⚙️ ขั้นตอนการตั้งค่าเวิร์กโฟลว์ (Workflow Setup)

### 1. การตั้งค่า Credential

คุณจะต้องตั้งค่า Credential ใน n8n สำหรับบริการเหล่านี้:

| Node ที่เกี่ยวข้อง | บริการ | Credential Type |
| :--- | :--- | :--- |
| `Google Drive search`, `Google Drive Download` | Google Drive | **OAuth2** (เพื่อเข้าถึง Drive) |
| `get image` | Google Drive API (สำหรับดึงไฟล์) | **OAuth2** (สำหรับ Google APIs ทั่วไป) |

### 2. การตั้งค่า Node เฉพาะ

| Node Name | การดำเนินการ | ค่าที่ต้องแก้ไข |
| :--- | :--- | :--- |
| **Schedule Trigger** | ตั้งเวลาทำงาน | แก้ไข `triggerAtHour` หากต้องการเปลี่ยนเวลาโพสต์อัตโนมัติ (ปัจจุบันคือ 9 โมง). |
| **Google Drive search** | ค้นหารูปภาพ | แก้ไข **Folder ID** (`1U_q2egTzc3eqgXcdxBln20DmH3qJLTrW`) ให้เป็น ID โฟลเดอร์ Google Drive ของคุณที่ใช้เก็บรูปภาพ. |
| **gemini image to Fabric type** | สร้างแคปชั่น | แทนที่ **`key`** ใน **Query Parameter** ด้วย **Gemini API Key** ของคุณ. |
| **post to facebook** | โพสต์ Facebook | 1. แก้ไข **`url`** โดยแทนที่ **`ใช้ ID เพจของเรา`** ด้วย Page ID ของคุณ. 2. แทนที่ **`value`** ของ `access_token` ด้วย **Facebook Page Access Token** ของคุณ. |

---

## 🗺️ แผนผังการทำงานของ Workflow (Flow Diagram)

เวิร์กโฟลว์นี้จะทำงานตามลำดับดังนี้:

1.  **Schedule Trigger** (`Schedule Trigger`) : เริ่มทำงานทุกวันเวลา 9:00 น.
2.  **Set date** : สร้างตัวแปรวันที่ปัจจุบันเพื่อใช้ในการค้นหาไฟล์ล่าสุด.
3.  **Google Drive search** : ค้นหารูปภาพที่ถูกอัปโหลดในโฟลเดอร์และถูกแก้ไขหลังเที่ยงคืนของวันนี้.
4.  **Google Drive Download** : ดาวน์โหลดข้อมูลเมตาของไฟล์ที่พบ.
5.  **Extract from File** : แปลงข้อมูลไฟล์ให้พร้อมใช้งานในรูปแบบ Binary.
6.  **gemini image to Fabric type** : ส่งรูปภาพและ Prompt ภาษาไทย (`ช่วยเขียนแคปชั่นเท่ๆ สำหรับโพสต์ขายสินค้าแฟชั่น...`) ไปยัง **Gemini API** เพื่อสร้างแคปชั่น.
7.  **get image** : ดึงไฟล์รูปภาพจริงจาก Google Drive อีกครั้งเพื่อเตรียมโพสต์.
8.  **post to facebook** : โพสต์รูปภาพ (จาก `get image`) พร้อมแคปชั่น (จากผลลัพธ์ของ `gemini image to Fabric type`) ไปยัง Facebook Page.
---
## Prompt ที่ใช้สำหรับ Gemini:
"ช่วยเขียนแคปชั่นเท่ๆ สำหรับโพสต์ขายสินค้าแฟชั่น บน Facebook พร้อมแฮชแท็ก โดยขอแค่ 1 คำตอบเท่านั้น"

---

## 🚀 เริ่มต้นใช้งาน (Getting Started)

1.  ดาวน์โหลดไฟล์ `.json` ของเวิร์กโฟลว์นี้.
2.  นำเข้า (Import) ไฟล์ไปยัง n8n Instance ของคุณ.
3.  ดำเนินการตามขั้นตอนในหัวข้อ **การตั้งค่า Node เฉพาะ** และ **การตั้งค่า Credential**.
4.  เปิดใช้งาน (Activate) เวิร์กโฟลว์.

🎉 **เสร็จสิ้น!** โพสต์ Facebook ของคุณจะถูกสร้างและโพสต์โดยอัตโนมัติทุกวัน.
