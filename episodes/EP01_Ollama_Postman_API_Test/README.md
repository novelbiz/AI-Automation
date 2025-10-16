# 🚀 การติดตั้ง Ollama และทดสอบ API ผ่าน Postman  
> อ้างอิงวิดีโอสอนจาก: [YouTube - ติดตั้ง Ollama และทดสอบ API ผ่าน Postman](https://www.youtube.com/watch?v=5fVw0u7uYZc)

---

🎥 วิดีโอสอน: 
[![Watch the video](https://github.com/novelbiz/AI_Automation/blob/main/assets/thumbnail/ep1.jpg)](https://www.youtube.com/watch?v=5fVw0u7uYZc)

---

## 🧠 Ollama คืออะไร  
**Ollama** คือแพลตฟอร์มสำหรับรันโมเดลภาษา (LLM) บนเครื่องของคุณเอง เช่น **Llama 3, Mistral, Phi-3, Gemma** ฯลฯ  
โดยไม่ต้องต่ออินเทอร์เน็ต เหมาะกับงาน local AI, automation และ integration กับเครื่องมืออย่าง Postman, n8n หรือ Python script  

---

## ⚙️ ขั้นตอนการติดตั้ง Ollama

### 🔹 1. ดาวน์โหลดและติดตั้ง Ollama  
- ไปที่เว็บไซต์หลัก: [https://ollama.com](https://ollama.com)  
- เลือกระบบปฏิบัติการของคุณ (Windows / macOS / Linux)  
- หลังติดตั้งเสร็จ ระบบจะเริ่ม **Ollama service** อัตโนมัติ  

ตรวจสอบว่า Ollama ทำงานแล้ว ด้วยคำสั่ง:
```bash
ollama --version
```

---

### 🔹 2. ดึงโมเดลมาทดสอบ (ตัวอย่าง: Llama 3)
รันคำสั่งใน Terminal หรือ CMD:
```bash
ollama pull llama3
```

รอจนโมเดลดาวน์โหลดเสร็จ จากนั้นลองรัน:
```bash
ollama run llama3
```

ระบบจะเข้าสู่โหมดสนทนา สามารถพิมพ์ข้อความทดสอบได้ เช่น  
```
Hello! What can you do?
```

---

### 🔹 3. ทดสอบ API ผ่าน Postman

1. เปิดโปรแกรม **Postman**
2. สร้าง Request ใหม่  
   - Method: `POST`  
   - URL: `http://localhost:11434/api/generate`

3. ไปที่แท็บ **Body → raw → JSON** แล้วใส่ตัวอย่าง payload:
```json
{
  "model": "llama3",
  "prompt": "เขียนสรุปสั้น ๆ เกี่ยวกับ AI คืออะไร"
}
```

4. กด **Send**  
   ระบบจะส่ง response กลับมาในรูปแบบ JSON เช่น:
```json
{
  "model": "llama3",
  "response": "AI หรือปัญญาประดิษฐ์ คือเทคโนโลยีที่ทำให้คอมพิวเตอร์สามารถเรียนรู้และตัดสินใจได้คล้ายมนุษย์..."
}
```

---

### 🔹 4. ตัวอย่าง Curl Command (ทดสอบผ่าน Terminal)
```bash
curl http://localhost:11434/api/generate -d '{
  "model": "llama3",
  "prompt": "สรุปแนวคิดของ Machine Learning"
}'
```

---

## 🧩 ปัญหาที่พบบ่อย

| ปัญหา | สาเหตุ | วิธีแก้ |
|--------|----------|----------|
| `connection refused` | Ollama service ยังไม่ทำงาน | รันคำสั่ง `ollama serve` |
| โหลดโมเดลช้า | อินเทอร์เน็ตหรือ server ช้า | ลองเปลี่ยน network หรือใช้โมเดลเล็กกว่า |
| Postman ไม่ตอบกลับ | Port ไม่ถูกต้อง | ตรวจสอบว่าใช้ `http://localhost:11434` |

---

## 🧠 ตัวอย่างการนำไปใช้
สามารถเชื่อมต่อ Ollama API กับระบบอื่น ๆ ได้ เช่น:
- **n8n Workflow Automation**
- **Python Script (requests / aiohttp)**
- **Frontend ผ่าน fetch()**

ตัวอย่าง Python:
```python
import requests

url = "http://localhost:11434/api/generate"
data = {
    "model": "llama3",
    "prompt": "อธิบายการทำงานของระบบนิเวศในป่า"
}

res = requests.post(url, json=data)
print(res.json())
```

---

## 🎯 สรุป
Ollama ช่วยให้คุณรันโมเดล AI บนเครื่องได้ง่าย ปลอดภัย และเชื่อมต่อกับ Postman หรือระบบ Automation ได้ทันที  
เหมาะสำหรับนักพัฒนา, นักวิจัย และผู้ที่ต้องการสร้าง AI assistant ส่วนตัวแบบ Local  

---

## 📺 แหล่งอ้างอิง
- [Ollama Official Website](https://ollama.com)  
- [Ollama API Documentation](https://github.com/ollama/ollama/blob/main/docs/api.md)  
- [YouTube: ติดตั้ง Ollama และทดสอบ API ผ่าน Postman](https://www.youtube.com/watch?v=5fVw0u7uYZc)
