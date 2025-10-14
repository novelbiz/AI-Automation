# 🤖 การยืนยันตัวตน OAuth 2.0 สำหรับ YouTube API ใน n8n

นี่คือคู่มือสำหรับนักพัฒนาที่ต้องการเชื่อมต่อ YouTube API กับ n8n ผ่าน OAuth 2.0 อย่างละเอียด

---

🎥 วิดีโอสอน: 
[![Watch the video](https://github.com/novelbiz/AI_Automation/blob/main/episodes/EP06_YouTubeAPI_OAuth2_Authen/Image/maxresdefault.webp)](https://youtu.be/p6I5f5g6BBk?si=kv9T1_QREqkOSsC7)

---

## 1. สร้างโปรเจกต์ใน Google Cloud Console

1. ไปที่ [Google Cloud Console](https://console.cloud.google.com/)
2. คลิกที่ **Select a project** แล้วเลือก **New Project**
3. ตั้งชื่อโปรเจกต์ เช่น `n8n-youtube-integration` แล้วคลิก **Create**

## 2. เปิดใช้งาน YouTube Data API v3

1. ในหน้าโปรเจกต์ของคุณ ไปที่ **APIs & Services > Library**
2. ค้นหา **YouTube Data API v3** แล้วคลิกที่มัน
3. คลิก **Enable** เพื่อเปิดใช้งาน API

## 3. ตั้งค่า OAuth Consent Screen

1. ไปที่ **APIs & Services > OAuth consent screen**
2. เลือก **External** แล้วคลิก **Create**
3. กรอกข้อมูลที่จำเป็น:

   * **App name**: ตั้งชื่อแอป เช่น `n8n YouTube Integration`
   * **User support email**: อีเมลที่ผู้ใช้สามารถติดต่อได้
   * **Developer contact email**: อีเมลของคุณ
4. คลิก **Save and Continue**

## 4. สร้าง OAuth 2.0 Credentials

1. ไปที่ **APIs & Services > Credentials**
2. คลิก **Create Credentials** แล้วเลือก **OAuth 2.0 Client IDs**
3. เลือก **Web application**
4. ในส่วน **Authorized redirect URIs** ให้เพิ่ม URI ต่อไปนี้:

   * `http://localhost:5678/rest/oauth2-credential/callback`
5. คลิก **Create**
6. บันทึก **Client ID** และ **Client Secret** ที่แสดงขึ้นมา

## 5. ตั้งค่า Credentials ใน n8n

1. ใน n8n ไปที่ **Settings > Credentials**
2. คลิก **New Credentials** แล้วเลือก **Google OAuth2 API**
3. กรอกข้อมูลที่จำเป็น:

   * **Client ID**: ที่ได้จากขั้นตอนก่อนหน้า
   * **Client Secret**: ที่ได้จากขั้นตอนก่อนหน้า
   * **Scopes**: เพิ่ม `https://www.googleapis.com/auth/youtube.upload`
4. คลิก **Save**

## 6. ยืนยันตัวตนผ่าน OAuth 2.0

1. ใน n8n ไปที่ **Credentials** ที่คุณสร้างขึ้น
2. คลิก **Authenticate**
3. ระบบจะเปิดหน้าเว็บให้คุณเข้าสู่ระบบ Google และอนุญาตการเข้าถึง
4. หลังจากอนุญาตแล้ว ระบบจะกลับมายัง n8n และแสดงข้อความว่า "Authentication successful"

## 7. สร้าง Workflow เพื่ออัปโหลดวิดีโอ

1. ใน n8n สร้าง Workflow ใหม่
2. เพิ่ม **YouTube** node
3. เลือก **Operation** เป็น `Upload Video`
4. กรอกข้อมูลที่จำเป็น:

   * **Video File**: เลือกไฟล์วิดีโอที่ต้องการอัปโหลด
   * **Title**: ตั้งชื่อวิดีโอ
   * **Description**: เพิ่มคำอธิบาย
   * **Tags**: เพิ่มแท็ก (ถ้ามี)
5. คลิก **Execute** เพื่อทดสอบ

## 8. แก้ไขปัญหาที่อาจเกิดขึ้น

* **ปัญหาการหมดอายุของ Refresh Token**: หากพบปัญหาการหมดอายุของ Refresh Token และต้องการยืนยันตัวตนใหม่บ่อย ๆ สามารถตั้งค่าให้แอปของคุณเป็น **Internal** ใน Google Cloud Console เพื่อหลีกเลี่ยงการหมดอายุของ token
* **ปัญหาการเชื่อมต่อใน n8n Desktop App**: หากใช้ n8n Desktop App และพบปัญหาการเชื่อมต่อ OAuth2 ให้ตรวจสอบว่าไฟล์ `n8n-desktop.env` มีค่าที่ถูกต้องสำหรับ `N8N_BASIC_AUTH_USER` และ `N8N_BASIC_AUTH_PASSWORD`

---

## 🎉 สรุป

คุณได้เรียนรู้วิธีการตั้งค่า OAuth 2.0 สำหรับ YouTube API ใน n8n อย่างละเอียดแล้วครับ!



