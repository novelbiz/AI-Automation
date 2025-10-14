# EP.2 สอนติดตั้งและทดสอบ n8n แบบ Step by Step

> 🎥 **วิดีโอต้นฉบับ:** [EP.2 | ระบบอัตโนมัติที่คุณก็ทำได้!](https://youtu.be/FIMOlQ31IOo)

---

## 📋 สารบัญ

1. [n8n คืออะไร?](#n8n-คืออะไร)
2. [ความต้องการของระบบ](#ความต้องการของระบบ)
3. [วิธีการติดตั้ง](#วิธีการติดตั้ง)
   - [ติดตั้งด้วย Docker (แนะนำ)](#ติดตั้งด้วย-docker-แนะนำ)
   - [ติดตั้งด้วย npm](#ติดตั้งด้วย-npm)
4. [การทดสอบและเริ่มใช้งาน](#การทดสอบและเริ่มใช้งาน)
5. [การแก้ปัญหาเบื้องต้น](#การแก้ปัญหาเบื้องต้น)
6. [ทรัพยากรเพิ่มเติม](#ทรัพยากรเพิ่มเติม)

---

## 🤖 n8n คืออะไร?

**n8n** (น่าจะอ่านว่า "nodemation") เป็นเครื่องมือระบบอัตโนมัติ (Workflow Automation) แบบ Open Source ที่ช่วยให้คุณเชื่อมต่อแอปพลิเคชันต่างๆ เข้าด้วยกัน โดยไม่ต้องเขียนโค้ดเยอะ

**จุดเด่นของ n8n:**
- 🆓 ใช้งานฟรี และเป็น Open Source
- 🎨 UI/UX ที่สวยงามและใช้งานง่าย
- 🔗 รองรับการเชื่อมต่อกับบริการมากกว่า 200+ บริการ
- 🏠 สามารถติดตั้งบนเครื่องตัวเอง (Self-hosted) ได้
- 🔐 ควบคุมข้อมูลของคุณเองได้เต็มที่

---

## 💻 ความต้องการของระบบ

### สำหรับการติดตั้งด้วย Docker:
- ✅ Docker Desktop หรือ Docker Engine
- ✅ หน่วยความจำ RAM อย่างน้อย 2GB
- ✅ พื้นที่ว่างบน Hard Drive อย่างน้อย 5GB

### สำหรับการติดตั้งด้วย npm:
- ✅ Node.js เวอร์ชัน 18.x, 20.x หรือ 22.x
- ✅ npm (มากับ Node.js อยู่แล้ว)
- ✅ หน่วยความจำ RAM อย่างน้อย 2GB

---

## 🚀 วิธีการติดตั้ง

### ติดตั้งด้วย Docker (แนะนำ)

Docker เป็นวิธีที่ง่ายที่สุดในการติดตั้ง n8n เพราะไม่ต้องกังวลเรื่องการติดตั้ง Dependencies ต่างๆ

#### **ขั้นตอนที่ 1: ติดตั้ง Docker**

**บน Windows/Mac:**
1. ดาวน์โหลด Docker Desktop จาก [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
2. ติดตั้งและเปิดโปรแกรม Docker Desktop
3. รอให้ Docker เริ่มทำงาน (จะเห็นไอคอนวาฬสีเขียว)

**บน Linux (Ubuntu/Debian):**
```bash
# อัปเดต package list
sudo apt update

# ติดตั้ง Docker
sudo apt install docker.io -y

# เริ่มการทำงานของ Docker
sudo systemctl start docker
sudo systemctl enable docker

# เพิ่ม user ปัจจุบันเข้ากลุ่ม docker (ไม่ต้องใช้ sudo)
sudo usermod -aG docker $USER
```

#### **ขั้นตอนที่ 2: รัน n8n ด้วย Docker**

เปิด Terminal (หรือ Command Prompt บน Windows) แล้วรันคำสั่ง:

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

**อธิบายคำสั่ง:**
- `-it` = รันในโหมด Interactive
- `--rm` = ลบ Container อัตโนมัติเมื่อปิด
- `--name n8n` = ตั้งชื่อ Container ว่า "n8n"
- `-p 5678:5678` = เปิดพอร์ต 5678 สำหรับเข้าถึง Web UI
- `-v ~/.n8n:/home/node/.n8n` = เก็บข้อมูลในเครื่องของคุณ
- `n8nio/n8n` = ชื่อ Docker Image

#### **ขั้นตอนที่ 3: รัน n8n แบบ Background (Production)**

ถ้าต้องการให้ n8n ทำงานตลอดเวลาในพื้นหลัง:

```bash
docker run -d \
  --name n8n \
  --restart unless-stopped \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

**คำสั่งเพิ่มเติมที่มีประโยชน์:**
```bash
# ดู logs ของ n8n
docker logs n8n -f

# หยุดการทำงาน
docker stop n8n

# เริ่มใหม่อีกครั้ง
docker start n8n

# ลบ Container
docker rm n8n
```

---

### ติดตั้งด้วย npm

เหมาะสำหรับผู้ที่ต้องการควบคุมการติดตั้งเองหรือต้องการพัฒนาต่อ

#### **ขั้นตอนที่ 1: ติดตั้ง Node.js**

1. ดาวน์โหลด Node.js จาก [https://nodejs.org](https://nodejs.org)
2. เลือกเวอร์ชัน LTS (Long Term Support) แนะนำเวอร์ชัน 20.x
3. ติดตั้งตามขั้นตอนปกติ

#### **ขั้นตอนที่ 2: ตรวจสอบการติดตั้ง**

เปิด Terminal แล้วรันคำสั่ง:

```bash
node --version
npm --version
```

ถ้าเห็นเวอร์ชันขึ้นมาแสดงว่าติดตั้งสำเร็จ

#### **ขั้นตอนที่ 3: ติดตั้ง n8n**

```bash
npm install -g n8n
```

**หมายเหตุ:** บน Linux/Mac อาจต้องใช้ `sudo`:
```bash
sudo npm install -g n8n
```

#### **ขั้นตอนที่ 4: เริ่มใช้งาน n8n**

```bash
n8n
```

หรือถ้าต้องการระบุพอร์ตเอง:

```bash
n8n start --tunnel
```

---

## ✅ การทดสอบและเริ่มใช้งาน

### **1. เปิดเว็บเบราว์เซอร์**

เข้าไปที่: **http://localhost:5678**

### **2. สร้างบัญชีผู้ใช้งาน**

ครั้งแรกที่เปิดจะให้คุณสร้างบัญชี:
- 📧 กรอกอีเมล
- 🔑 กรอกรหัสผ่าน
- 👤 กรอกชื่อ (ถ้ามี)

### **3. ทดสอบ Workflow แรก**

#### **สร้าง Workflow ทดสอบง่ายๆ:**

1. คลิกปุ่ม **"+ Create Workflow"**
2. เพิ่ม Node แรก: **"Manual Trigger"** (กดเพื่อเริ่มต้น)
3. เพิ่ม Node ที่สอง: **"Set"** (กำหนดค่าตัวแปร)
   - ตั้งค่า: `name` = `Hello n8n!`
4. เชื่อมต่อ Node ทั้งสอง
5. กดปุ่ม **"Execute Workflow"**

ถ้าเห็นผลลัพธ์แสดงว่าใช้งานได้แล้ว! 🎉

---

## 🔧 การแก้ปัญหาเบื้องต้น

### **ปัญหา: เปิด localhost:5678 ไม่ได้**

**วิธีแก้:**
1. ตรวจสอบว่า Docker หรือ n8n ทำงานอยู่หรือไม่
   ```bash
   # สำหรับ Docker
   docker ps
   
   # สำหรับ npm
   ps aux | grep n8n
   ```

2. ตรวจสอบว่าพอร์ต 5678 ไม่ถูกใช้งานโดยโปรแกรมอื่น
   ```bash
   # บน Windows
   netstat -ano | findstr :5678
   
   # บน Mac/Linux
   lsof -i :5678
   ```

3. ลองเปลี่ยนพอร์ต:
   ```bash
   # Docker
   docker run -p 5679:5678 n8nio/n8n
   
   # npm
   n8n start --port 5679
   ```

### **ปัญหา: Docker ใช้งานไม่ได้บน Windows**

**วิธีแก้:**
1. เปิดใช้งาน Virtualization ใน BIOS
2. เปิดใช้งาน WSL 2 (Windows Subsystem for Linux)
   ```powershell
   wsl --install
   ```

### **ปัญหา: หน่วยความจำเต็ม**

**วิธีแก้:**
- จำกัด memory ที่ Docker ใช้:
  ```bash
  docker run -m 1g n8nio/n8n
  ```

---

## 📚 ทรัพยากรเพิ่มเติม

### **เอกสารอย่างเป็นทางการ:**
- 📖 [n8n Documentation](https://docs.n8n.io/)
- 🛠️ [n8n Community Forum](https://community.n8n.io/)
- 💻 [n8n GitHub Repository](https://github.com/n8n-io/n8n)

### **ช่องทางการเรียนรู้:**
- 🎥 [YouTube Channel - ระบบอัตโนมัติที่คุณก็ทำได้!](https://www.youtube.com/@YourChannel)
- 📝 [n8n Workflow Templates](https://n8n.io/workflows)

### **คอมมูนิตี้ไทย:**
- 💬 [Facebook Group - n8n Thailand](https://facebook.com/groups/n8nthailand) (ตัวอย่าง)
- 🗨️ [Discord Server](https://discord.gg/n8n) (สากล)

---

## 📥 Download

### **ลิงก์ดาวน์โหลดที่จำเป็น:**

| เครื่องมือ | ลิงก์ดาวน์โหลด | หมายเหตุ |
|----------|--------------|---------|
| Docker Desktop | [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop) | สำหรับ Windows/Mac |
| Node.js | [https://nodejs.org](https://nodejs.org) | แนะนำ LTS 20.x |
| n8n (npm) | `npm install -g n8n` | ใช้คำสั่งนี้ใน Terminal |

---

## 🎯 สิ่งที่จะได้เรียนรู้ในตอนต่อไป

- 🔌 การเชื่อมต่อกับบริการต่างๆ (Google Sheets, Discord, Line, etc.)
- 📨 สร้างระบบแจ้งเตือนอัตโนมัติ
- 🤖 สร้าง Chatbot ด้วย n8n
- 📊 ดึงข้อมูลและวิเคราะห์ผลอัตโนมัติ
- ⏰ ตั้งเวลาให้ Workflow ทำงานเอง (Cron Jobs)

---

## ⭐ สนับสนุนช่อง

ถ้าชอบเนื้อหานี้ อย่าลืม:
- 👍 กด Like
- 🔔 Subscribe และกดกระดิ่งเพื่อไม่พลาดคลิปใหม่
- 💬 Comment บอกว่าอยากเห็นเนื้อหาอะไรต่อไป
- 🔗 แชร์ให้เพื่อนๆ ที่สนใจ

---

> 💡 **เคล็ดลับ:** หลังจากติดตั้งเสร็จแล้ว ลองเล่นกับ Templates ที่มีให้ในหน้า Workflows เพื่อเรียนรู้การทำงานของ n8n ได้รวดเร็วขึ้น!
