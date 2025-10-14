# EP03_n8n_WebScraping_to_WordPress

> Workflow ตัวอย่างสำหรับ **ดึงข้อมูลจากเว็บ** -> วิเคราะห์/ปรับแต่ง -> **โพสต์ขึ้น WordPress อัตโนมัติ**  
> ใช้ร่วมกับ AI (ในตัวอย่างใช้ Google Gemini / agent) เพื่อเขียนเนื้อหาและจัดโครงเรื่อง  

---

## 🎯 จุดประสงค์

- ดึงข้อมูล (เช่น รูป, ชื่อข่าว, เนื้อหา) จากเว็บไซต์ที่ตั้งไว้  
- ประมวลผลเนื้อหา (ผ่าน Agent / AI) ให้เหมาะกับรูปแบบบทความ  
- อัปโหลดรูปภาพไปยัง WordPress Media  
- สร้างโพสต์ใหม่ใน WordPress พร้อมตั้ง Featured Image / metadata  
- อัปเดตโพสต์ (update metadata) ถ้าจำเป็น  

---

## 🚀 วิธีใช้งาน (Step by Step)

1. ดาวน์โหลดไฟล์ `workflow.json`  
2. เข้า n8n → กด **Import from File** → เลือก `workflow.json`  
3. ตรวจสอบ node แต่ละตัว:  
   - ใส่ **Credentials** ให้กับ WordPress (httpBasicAuth)  
   - ใส่ **Credentials** ให้กับ Google Gemini / AI agent  
4. ปรับ URL จุดต่าง ๆ (เช่น URL สำหรับโพสต์, API endpoint) ให้ตรงกับโดเมน WordPress ของคุณ  
5. ถ้าต้องการ ตั้งให้ workflow ทำงานอัตโนมัติ → ตรวจสอบ node **Schedule Trigger**  
6. ทดสอบรัน workflow ด้วยมือก่อนสั่งให้รันอัตโนมัติ  
7. ตรวจสอบใน WordPress ว่าโพสต์ขึ้น/รูปถูกอัปโหลดหรือไม่  

---

## 🧩 โครงสร้าง Node & ลำดับการทำงาน

| Node | ความหมาย / หน้าที่ | จุดที่ควรตรวจสอบ |
|------|---------------------|--------------------|
| **Schedule Trigger** | ตั้งเวลาให้ workflow ทำงาน | ถ้าจะรันอัตโนมัติ ต้องตั้ง interval ให้เหมาะสม |
| **Set URL** | กำหนด URL ของเว็บไซต์ที่ต้องการดึงข้อมูล | ปรับชื่อ field / ค่า URL ตามเว็บเป้าหมาย |
| **Fetch Website (HTTP Request)** | ดึง HTML / ข้อมูลเว็บ | timeout, responseFormat, headers |
| **HTML (extractHtmlContent)** | ดึงส่วนที่ต้องการจาก HTML (รูป, ชื่อข่าว ฯลฯ) | ตรวจสอบ cssSelector ที่เลือกให้ตรงกับโครงสร้างเว็บ |
| **Aggregate** | รวบรวมข้อมูลหลายแถวให้เป็นชุด | ตรวจว่า fieldToAggregate ครอบคลุมข้อมูลที่ต้องการ |
| **AI Agent Node** | สร้างเนื้อหา / ประมวลผลข้อความ | ใส่ prompt, เชื่อม API, กำหนด schema output |
| **HTTP Request → Upload to WordPress (media)** | อัปโหลดไฟล์รูปไปยัง WordPress Media | ตรวจ credentials, headers, content-disposition, mimeType |
| **Set Post Data** | จัดรูป JSON สำหรับโพสต์ เช่น title, content, status, categories | ตรวจว่า content มี HTML ถูกต้อง |
| **HTTP Request → Post to WordPress (posts)** | สร้างโพสต์ใหม่ | ตรวจ credentials, endpoint, body parameter |
| **HTTP Request → Update Meta Data** | อัปเดตค่าของโพสต์ เช่น featured_media | ตรวจว่า id ของ media ถูกอัปเดต |

---

## 🛡️ หมายเหตุความปลอดภัย

- ไฟล์ `workflow.json` ที่ให้มา **ได้ถูก sanitized แล้ว** — ไม่มี credential id หรือ key ใด ๆ  
- ก่อนใช้งานจริงให้ **ตั้ง credential ใหม่** ใน instance n8n ของคุณ  
- อย่าเผย credential / API key บน GitHub หรือที่สาธารณะ  
- ตรวจสอบว่าเว็บไซต์ที่ดึงข้อมูลยินยอมให้ scraping (ทางกฎหมาย / policy)  

---

## 📦 ดาวน์โหลด & Import

1. ดาวน์โหลดไฟล์ `workflow.json` (ไฟล์ sanitized)  
2. ไปที่ n8n → เลือก **Workflow → Import from File** → เลือกไฟล์ที่ดาวน์โหลดมา  
3. ตั้ง credential ใหม่ตามที่อธิบายด้านบน  

---

## 📺 วิดีโอสอนประกอบ

สามารถดูวิธีทำแบบ step-by-step ได้จากวิดีโอ:  
[EP03 n8n WebScraping to WordPress](https://www.youtube.com/watch?v=M_UnuPLWCfk)  

---  
### ✍ สรุป

นี่คือ workflow ตัวอย่างที่ช่วยให้คุณเริ่มต้นระบบ **Web Scraping → AI ประมวลผล → โพสต์อัตโนมัติบน WordPress** ได้ทันที ปรับแต่งให้ตรงกับเว็บเป้าหมายของคุณ แล้วทดสอบให้มั่นใจก่อนใช้งานจริง 😊  


