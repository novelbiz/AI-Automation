# โปรแกรมอ่านบัตรประชาชนไทย (Thai National ID Card Reader)

โปรแกรม Command-line สำหรับอ่านข้อมูลจากบัตรประชาชนไทยผ่านเครื่องอ่านบัตรอัจฉริยะ (Smart Card Reader)

**พัฒนาโดย:** Novelbiz (https://novelbiz.co.th)  
**License:** MIT License 2025

---

## 📋 คุณสมบัติ (Features)

โปรแกรมสามารถอ่านข้อมูลจากบัตรประชาชนไทยได้ดังนี้:
- เลขบัตรประชาชน 13 หนัก
- ชื่อ-นามสกุล (ภาษาไทยและอังกฤษ)
- วันเกิด (แสดงทั้งรูปแบบไทยและอังกฤษ)
- เพศ
- วันที่ออกบัตร
- วันที่หมดอายุ
- สถานที่ออกบัตร
- ที่อยู่ตามทะเบียนบ้าน
- หมายเลขคำขอ

---

## 💻 ความต้องการของระบบ (System Requirements)

### ฮาร์ดแวร์
- เครื่องอ่านบัตรอัจฉริยะ (Smart Card Reader) ที่รองรับมาตรฐาน ISO 7816
- พอร์ต USB สำหรับเชื่อมต่อเครื่องอ่านบัตร

### ซอฟต์แวร์
- **ระบบปฏิบัติการ:** Windows 10/11 (แนะนำ) หรือ Linux/macOS
- **Python:** เวอร์ชัน 3.7 ขึ้นไป
- **Smart Card Service:** ต้องเปิดใช้งาน Smart Card Service บน Windows

---

## 🔧 การติดตั้ง (Installation)

### ขั้นตอนที่ 1: ติดตั้ง Python

ดาวน์โหลดและติดตั้ง Python จาก [python.org](https://www.python.org/downloads/)

ตรวจสอบการติดตั้ง:
```bash
python --version
```

### ขั้นตอนที่ 2: ติดตั้ง Driver สำหรับ Smart Card Reader

ติดตั้ง driver ของเครื่องอ่านบัตรตามคู่มือของผู้ผลิต

### ขั้นตอนที่ 3: เปิดใช้งาน Smart Card Service (สำหรับ Windows)

1. กดปุ่ม `Win + R` พิมพ์ `services.msc` แล้วกด Enter
2. หาบริการชื่อ **Smart Card**
3. คลิกขวาเลือก **Properties**
4. ตั้งค่า Startup type เป็น **Automatic**
5. คลิก **Start** เพื่อเริ่มบริการ
6. คลิก **OK**

### ขั้นตอนที่ 4: ติดตั้ง Library ที่จำเป็น

#### สำหรับ Windows:
```bash
pip install pyscard
```

#### สำหรับ Linux (Ubuntu/Debian):
```bash
# ติดตั้ง dependencies
sudo apt-get update
sudo apt-get install pcscd pcsc-tools libpcsclite-dev swig

# ติดตั้ง Python library
pip install pyscard
```

#### สำหรับ macOS:
```bash
# ติดตั้ง dependencies ผ่าน Homebrew
brew install swig

# ติดตั้ง Python library
pip install pyscard
```

### ขั้นตอนที่ 5: ดาวน์โหลดโปรแกรม

```bash
# ดาวน์โหลดไฟล์ หรือ clone repository
git clone [repository-url]
cd Thai_id_card
```

หรือคัดลอกไฟล์ `Thai_id_card_cli.py` ไปยังโฟลเดอร์ที่ต้องการ

---

## 🚀 วิธีใช้งาน (Usage)

### ขั้นตอนการใช้งาน

1. เชื่อมต่อเครื่องอ่านบัตรเข้ากับคอมพิวเตอร์
2. เปิด Command Prompt หรือ Terminal
3. ไปยังโฟลเดอร์ที่มีไฟล์โปรแกรม
4. รันคำสั่ง:

```bash
python Thai_id_card_cli.py
```

5. ใส่บัตรประชาชนลงในเครื่องอ่านบัตร
6. รอให้โปรแกรมอ่านข้อมูล
7. ข้อมูลจะแสดงบนหน้าจอ

### ตัวอย่างผลลัพธ์

```
======================================================================
               โปรแกรมอ่านบัตรประชาชนไทย - Novelbiz
                         MIT License 2025
======================================================================
[เริ่มต้น] โปรแกรมพร้อมทำงาน
======================================================================

[สถานะ] พบเครื่องอ่านบัตร: ACS ACR122U PICC Interface 0

[พร้อม] ใส่บัตรประชาชนเพื่ออ่านข้อมูล...

======================================================================
[เริ่มอ่าน] กำลังอ่านข้อมูลบัตรประชาชน...
======================================================================

[เชื่อมต่อ] เชื่อมต่อกับ: ACS ACR122U PICC Interface 0
[ATR] 3B 67 00 00 ...
[สำเร็จ] เลือก Thai ID Applet สำเร็จ (SW: 90 00)

----------------------------------------------------------------------
                         ข้อมูลบัตรประชาชน
----------------------------------------------------------------------

📌 เลขบัตรประชาชน: 1234567890123
👤 ชื่อ-นามสกุล (ไทย): นาย สมชาย ใจดี
👤 ชื่อ-นามสกุล (อังกฤษ): Mr. Somchai Jaidee
🎂 วันเกิด (ไทย): 15 มกราคม 2533
🎂 วันเกิด (อังกฤษ): 15 January 1990
⚧ เพศ: ชาย

📅 วันที่ออกบัตร (ไทย): 1 มกราคม 2563
📅 วันที่ออกบัตร (อังกฤษ): 1 January 2020
📅 วันที่หมดอายุ (ไทย): 31 ธันวาคม 2573
📅 วันที่หมดอายุ (อังกฤษ): 31 December 2030
🏢 สถานที่ออกบัตร: สำนักทะเบียนอำเภอเมือง จังหวัดกรุงเทพมหานคร

🏠 ที่อยู่: 123 ถนนสุขุมวิท แขวงคลองเตย เขตคลองเตย กรุงเทพมหานคร 10110
🔢 หมายเลขคำขอ: AA1234567890

======================================================================
[สำเร็จ] ✅ อ่านข้อมูลบัตรประชาชนเรียบร้อยแล้ว
======================================================================

[เสร็จสิ้น] อ่านข้อมูลเสร็จสมบูรณ์
```

---

## ❗ การแก้ปัญหา (Troubleshooting)

### ปัญหา: ไม่พบเครื่องอ่านบัตร

**แก้ไข:**
- ตรวจสอบว่าเครื่องอ่านบัตรเชื่อมต่อกับคอมพิวเตอร์แล้ว
- ติดตั้ง driver ของเครื่องอ่านบัตร
- ตรวจสอบว่า Smart Card Service ทำงานอยู่ (Windows)

### ปัญหา: ไม่สามารถติดตั้ง pyscard ได้

**แก้ไข:**
```bash
# ลองอัปเดต pip ก่อน
python -m pip install --upgrade pip

# สำหรับ Windows อาจต้องติดตั้ง Visual C++ Build Tools
# ดาวน์โหลดจาก: https://visualstudio.microsoft.com/visual-cpp-build-tools/

# ลองติดตั้งอีกครั้ง
pip install pyscard
```

### ปัญหา: Smart Card Service ไม่ทำงาน (Windows)

**แก้ไข:**
```bash
# เปิด Command Prompt แบบ Administrator แล้วรันคำสั่ง:
sc start SCardSvr
```

### ปัญหา: ไม่สามารถอ่านข้อมูลจากบัตรได้

**แก้ไข:**
- ทำความสะอาดบัตรและเครื่องอ่านบัตร
- ตรวจสอบว่าบัตรใส่ถูกทิศทาง
- ลองถอดบัตรแล้วใส่ใหม่
- รีสตาร์ทโปรแกรม

### ปัญหา: ข้อความแสดงผลผิดเพี้ยน

**แก้ไข:**
- ตรวจสอบว่า Terminal รองรับ UTF-8 และ TIS-620
- สำหรับ Windows ให้ตั้งค่า Code Page:
```bash
chcp 65001
```

---

## 📝 หมายเหตุ

- โปรแกรมนี้ออกแบบมาสำหรับบัตรประชาชนไทยเท่านั้น
- ข้อมูลที่อ่านได้จะแสดงบนหน้าจอเท่านั้น ไม่มีการบันทึกลงไฟล์
- การอ่านข้อมูลต้องปฏิบัติตามกฎหมายและระเบียบที่เกี่ยวข้อง
- ควรใช้โปรแกรมนี้เพื่อวัตถุประสงค์ที่ถูกต้องตามกฎหมายเท่านั้น

---

## 📄 License

โปรแกรมนี้เผยแพร่ภายใต้ MIT License

```
Copyright 2025 NOVELBIZ CO., LTD.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

---

## 👨‍💻 ผู้พัฒนา

**Novelbiz Co., Ltd.**
- Website: https://novelbiz.co.th
- Email: support@novelbiz.co.th

---

## 🤝 การสนับสนุน

หากพบปัญหาหรือต้องการสอบถามข้อมูลเพิ่มเติม กรุณาติดต่อผ่าน:
- Website: https://novelbiz.co.th
- หรือสร้าง Issue ใน GitHub Repository

---

## 📚 เอกสารอ้างอิง

- [pyscard Documentation](https://pyscard.sourceforge.io/)
- [PC/SC Workgroup](https://www.pcscworkgroup.com/)
- [ISO/IEC 7816 Standard](https://www.iso.org/standard/54089.html)

---

**เวอร์ชัน:** 1.0.0  
**อัปเดตล่าสุด:** 2025
