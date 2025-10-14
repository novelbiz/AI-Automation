<!doctype html>
<html lang="th">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>ติดตั้ง Ollama และทดสอบ API ผ่าน Postman — README</title>
  <style>
    :root{
      --bg:#0f1724; --card:#0b1220; --muted:#94a3b8; --accent:#60a5fa; --code-bg:#071026;
      --success:#10b981; --danger:#ef4444; --glass: rgba(255,255,255,0.03);
    }
    body{font-family:Inter, "Segoe UI", Roboto, "Helvetica Neue", Arial; background:linear-gradient(180deg,#071025 0%, #08122a 100%); color:#e6eef8; margin:0; padding:32px;}
    .wrapper{max-width:980px; margin:0 auto;}
    header{display:flex; align-items:center; gap:16px; margin-bottom:20px;}
    header img{width:56px; height:56px; border-radius:10px; background:var(--glass); padding:8px;}
    h1{margin:0; font-size:24px;}
    p.lead{color:var(--muted); margin-top:6px;}
    .card{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)); border-radius:12px; padding:20px; box-shadow:0 6px 18px rgba(2,6,23,0.6); margin-bottom:18px;}
    h2{color:#dff3ff; margin-top:0;}
    pre{background:var(--code-bg); padding:14px; border-radius:8px; overflow:auto; color:#cfefff; font-family: "Courier New", monospace; font-size:13px;}
    code{background:rgba(255,255,255,0.03); padding:2px 6px; border-radius:6px;}
    .note{background: linear-gradient(90deg, rgba(96,165,250,0.06), rgba(16,185,129,0.03)); padding:10px; border-radius:8px; color:var(--muted); margin:10px 0;}
    .grid{display:grid; grid-template-columns:1fr 1fr; gap:12px;}
    .btn{display:inline-block; padding:8px 12px; border-radius:8px; background:var(--accent); color:#042028; text-decoration:none; font-weight:600;}
    footer{color:var(--muted); font-size:13px; margin-top:18px; text-align:center;}
    details{background:rgba(255,255,255,0.01); padding:10px; border-radius:8px;}
    ul.ol-post{padding-left:20px; color:var(--muted);}
    @media(max-width:760px){ .grid{grid-template-columns:1fr; } header{flex-direction:row;} }
  </style>
</head>
<body>
  <div class="wrapper">
    <header>
      <img alt="Ollama" src="https://ollama.com/favicon.ico" />
      <div>
        <h1>ติดตั้ง Ollama และทดสอบ API ผ่าน Postman</h1>
        <p class="lead">คู่มือสั้น ๆ ที่ทำตามได้จริง — เอาไปใส่ใน GitHub เป็น `README.html` ได้เลย</p>
      </div>
    </header>

    <section class="card">
      <h2>ภาพรวม</h2>
      <p>Ollama เป็น local AI runtime ที่ช่วยให้คุณรันโมเดล (เช่น Llama3, Mistral ฯลฯ) บนเครื่องของคุณและเรียกผ่าน REST API บนพอร์ต <code>11434</code> — เหมาะสำหรับการทดสอบและพัฒนาแบบไม่ต้องพึ่งคลาวด์</p>
      <p class="note">อ้างอิงวิดีโอ (ต้นแบบ): <a href="https://www.youtube.com/watch?v=5fVw0u7uYZc" target="_blank" rel="noopener" style="color:var(--accent)">YouTube - Ollama API Setup & Test</a></p>
    </section>

    <section class="card">
      <h2>ขั้นตอนติดตั้งอย่างย่อ</h2>
      <ol class="ol-post">
        <li>ดาวน์โหลด installer จาก <a href="https://ollama.com/download" target="_blank" rel="noopener" style="color:var(--accent)">ollama.com/download</a></li>
        <li>ติดตั้ง (Windows / macOS / Linux ตามตัวติดตั้งที่ให้มา)</li>
        <li>ตรวจสอบโดยรัน: <code>ollama --version</code></li>
        <li>ดึงโมเดลที่ต้องการ (ตัวอย่าง: <code>ollama pull llama3</code>)</li>
        <li>ทดสอบเรียก API ที่ <code>http://localhost:11434</code></li>
      </ol>
    </section>

    <section class="card">
      <h2>คำสั่งพื้นฐาน (Terminal)</h2>
      <pre><code># ตรวจสอบเวอร์ชัน
ollama --version

# ดูโมเดลที่มีในเครื่อง
ollama list

# ดึงโมเดล (ตัวอย่าง: llama3)
ollama pull llama3

# รันโมเดลแบบ interactive (ทดลองพิมพ์ prompt แล้วดูผล)
ollama run llama3

# ลบโมเดลเมื่อไม่ต้องการ
ollama rm llama3

# เปิด server (ดู log ถ้าต้องการ)
ollama serve
</code></pre>
      <p class="note">ถ้าคุณเห็นพอร์ต <code>11434</code> แสดงว่า API พร้อมเรียกใช้งาน</p>
    </section>

    <section class="card">
      <h2>ทดสอบ API ด้วยเบราว์เซอร์ (quick check)</h2>
      <p>เปิด URL นี้เพื่อตรวจสอบว่าเซิร์ฟเวอร์ตอบ:</p>
      <pre><code>http://localhost:11434/api/tags</code></pre>
      <p>ผลที่คาดหวังเป็น JSON แบบย่อ เช่น:</p>
      <pre><code>{
  "models": [
    { "name": "llama3" }
  ]
}</code></pre>
    </section>

    <section class="card">
      <h2>ตัวอย่างการทดสอบผ่าน Postman</h2>
      <p>สร้าง <strong>POST</strong> request ดังนี้:</p>
      <ul class="ol-post">
        <li><strong>URL:</strong> <code>http://localhost:11434/api/generate</code></li>
        <li><strong>Headers:</strong> <code>Content-Type: application/json</code></li>
        <li><strong>Body (raw, JSON):</strong></li>
      </ul>

      <pre><code>{
  "model": "llama3",
  "prompt": "อธิบายว่า AI คืออะไร แบบสั้นๆ และเข้าใจง่าย"
}</code></pre>

      <p>กด <code>Send</code> — คุณจะได้รับการตอบกลับ (อาจเป็น streaming ถ้าโมเดลตอบแบบนั้น)</p>

      <details>
        <summary style="cursor:pointer; font-weight:600; color:var(--accent)">ตัวอย่าง Postman Collection (JSON) — นำไป Import ได้</summary>
        <pre><code>{
  "info": {
    "name": "Ollama API - Quick Test",
    "_postman_id": "ollama-collection-example",
    "description": "Import this into Postman to test Ollama local API",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "Generate - llama3",
      "request": {
        "method": "POST",
        "header": [
          {
            "key": "Content-Type",
            "value": "application/json"
          }
        ],
        "body": {
          "mode": "raw",
          "raw": "{\n  \"model\": \"llama3\",\n  \"prompt\": \"อธิบายว่า AI คืออะไร แบบสั้นๆ และเข้าใจง่าย\"\n}"
        },
        "url": {
          "raw": "http://localhost:11434/api/generate",
          "protocol": "http",
          "host": ["localhost"],
          "port": "11434",
          "path": ["api","generate"]
        }
      },
      "response": []
    },
    {
      "name": "List models",
      "request": {
        "method": "GET",
        "header": [],
        "url": {
          "raw": "http://localhost:11434/api/tags",
          "protocol": "http",
          "host": ["localhost"],
          "port": "11434",
          "path": ["api","tags"]
        }
      }
    }
  ]
}</code></pre>
      </details>

      <p class="note">นำ JSON ข้างต้นไป <strong>Import</strong> ใน Postman → เลือก <em>File → Import</em> → วาง JSON หรืออัปโหลดไฟล์</p>
    </section>

    <section class="card">
      <h2>ตัวอย่าง cURL</h2>
      <pre><code>curl -X POST "http://localhost:11434/api/generate" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llama3",
    "prompt": "Hello from curl! Explain AI in one paragraph."
  }'
</code></pre>
      <p class="note">ใช้เมื่อต้องการทดสอบแบบไม่ใช้ GUI</p>
    </section>

    <section class="card">
      <h2>โครงสร้างไฟล์ตัวอย่างสำหรับ GitHub</h2>
      <pre><code>ollama-api-demo/
├─ README.html         # ไฟล์นี้ (HTML) สำหรับ GitHub pages หรืออ่านบนเว็บ
├─ README.md           # (ตัวเลือก) เวอร์ชัน markdown
├─ postman/
│  └─ ollama_api_test.json   # Postman collection (นำเข้าได้เลย)
└─ examples/
   └─ generate_with_curl.sh
</code></pre>
    </section>

    <section class="card">
      <h2>เคล็ดลับและข้อควรระวัง</h2>
      <ul class="ol-post">
        <li>ถ้าพอร์ต 11434 ถูกใช้งานอยู่ ให้ตรวจสอบ process หรือเปลี่ยนพอร์ตใน config ของ Ollama (หากมีตัวเลือก)</li>
        <li>เมื่อดึงโมเดลขนาดใหญ่ อาจใช้เวลาและพื้นที่ดิสก์มาก — ตรวจสอบพื้นที่ก่อน pull</li>
        <li>สำหรับการใช้งานจริงในระบบพัฒนา แนะนำแยกสิทธิ์และ firewall ให้เหมาะสม (อย่าเปิดให้เข้าถึงจากอินเทอร์เน็ตสาธารณะโดยไม่ตั้งค่า)</li>
      </ul>
    </section>

    <section class="card">
      <h2>สรุปสั้น ๆ</h2>
      <p>ติดตั้ง Ollama → ดึงโมเดล → ทดสอบ CLI → ใช้ Postman หรือ cURL เรียก <code>/api/generate</code> ที่ <code>http://localhost:11434</code> — เสร็จเรียบร้อย! 🎉</p>
    </section>

    <footer>
      <div>เขียนโดย: AI Mentor (เป็นกันเอง + ตลกนิด ๆ) — ส่งต่อให้บิ้กใช้งานได้เลย</div>
      <div style="margin-top:8px">อยากให้ผมสร้างไฟล์ `ollama_api_test.json` ให้ดาวน์โหลดจริง ๆ หรือจะให้ผมแปลงเป็น <code>README.md</code> ด้วย? ผมพร้อมจัดให้</div>
    </footer>
  </div>
</body>
</html>
