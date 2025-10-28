# 📊 EP. 23 ไม่พลาดโอกาสขาย! สร้างด้วย n8n วิเคราะห์ยอดขายรายวัน + Forecast ส่งเข้า LINE และ Email

## 📋 สารบัญ
1. [ภาพรวม Workflow](#ภาพรวม-workflow)
2. [สถาปัตยกรรมและการทำงาน](#สถาปัตยกรรมและการทำงาน)
3. [รายละเอียดแต่ละ Node](#รายละเอียดแต่ละ-node)
4. [การตั้งค่าและ Prerequisites](#การตั้งค่าและ-prerequisites)
5. [วิธีการติดตั้งและใช้งาน](#วิธีการติดตั้งและใช้งาน)
6. [การแก้ไขปัญหา](#การแก้ไขปัญหา)

---

## 🎯 ภาพรวม Workflow

Workflow นี้เป็นระบบรายงานยอดขายสินค้า Apple อัตโนมัติที่ทำงานทุกวันเวลา 08:00 น. โดยจะดึงข้อมูลจาก Google Sheets วิเคราะห์ด้วย AI และส่งรายงานผ่าน Email และ LINE

### ✨ ฟีเจอร์หลัก
- ⏰ รันอัตโนมัติทุกเช้าเวลา 08:00 น.
- 📊 สร้างกราฟ 3 แบบ: Bar Chart, Line Chart, Pie Chart
- 🤖 วิเคราะห์ข้อมูลด้วย Google Gemini AI
- 📧 ส่งรายงานผ่าน Gmail และ LINE
- 📈 วิเคราะห์แนวโน้มและให้คำแนะนำเชิงกลยุทธ์

### 🔄 ขั้นตอนการทำงาน
```
Schedule Trigger (8:00 AM)
    ↓
Get row(s) in sheet
    ↓
Process Sales Data
    ↓
Generate Charts
    ↓
Basic LLM Chain (Google Gemini)
    ↓
    ├─→ Format Email Message → Send a message (Gmail)
    └─→ Format LINE Message → Send via LINE Messaging API
```

---

## 🏗️ สถาปัตยกรรมและการทำงาน

### การไหลของข้อมูล (Data Flow)

1. **Data Collection** - ดึงข้อมูลจาก Google Sheets
2. **Data Processing** - ประมวลผลและคำนวณสถิติ
3. **Visualization** - สร้างกราฟด้วย QuickChart API
4. **AI Analysis** - วิเคราะห์ด้วย Google Gemini
5. **Distribution** - ส่งรายงานผ่านหลายช่องทาง

---

## 📦 รายละเอียดแต่ละ Node

### 1. Schedule Trigger (8:00 AM)

**ประเภท:** `n8n-nodes-base.scheduleTrigger`  
**Version:** 1.2

#### หน้าที่
เป็น trigger node ที่ใช้เริ่มต้น workflow อัตโนมัติทุกวัน

#### การตั้งค่า
```json
{
  "cronExpression": "0 8 * * *"
}
```

#### คำอธิบาย Cron Expression
- `0` = นาทีที่ 0
- `8` = ชั่วโมงที่ 8 (08:00 น.)
- `*` = ทุกวันในเดือน
- `*` = ทุกเดือน
- `*` = ทุกวันในสัปดาห์

#### การปรับแต่ง
หากต้องการเปลี่ยนเวลา สามารถแก้ไข cron expression ได้ เช่น:
- `0 9 * * *` = 09:00 น. ทุกวัน
- `0 8 * * 1` = 08:00 น. ทุกวันจันทร์
- `0 8 1 * *` = 08:00 น. วันที่ 1 ของทุกเดือน

---

### 2. Get row(s) in sheet

**ประเภท:** `n8n-nodes-base.googleSheets`  
**Version:** 4.7

#### หน้าที่
ดึงข้อมูลทั้งหมดจาก Google Sheets ที่บันทึกยอดขายสินค้า Apple

#### การตั้งค่า
```javascript
{
  "documentId": "1DkT4MRxtGPIIqOuofyAcO5nMaAwn-WREVkHbzdsASyU",
  "sheetName": "การตอบแบบฟอร์ม 1" (ID: 1437698346)
}
```

#### โครงสร้างข้อมูลใน Sheet
คาดว่ามีคอลัมน์ดังนี้:
- **วันที่ขาย** - วันที่เกิดการขาย
- **สินค้า** - ชื่อผลิตภัณฑ์ Apple
- **จำนวน** - จำนวนสินค้าที่ขาย (ชิ้น)
- **ยอดขายรวม** - มูลค่ารวม (บาท)
- **หมวดหมู่** - หมวดหมู่สินค้า (เช่น iPhone, iPad, Mac)
- **ประทับเวลา** - เวลาที่บันทึกข้อมูล (รูปแบบ: DD/MM/YYYY, HH:MM:SS)

#### Credentials ที่ต้องการ
- **Google Sheets OAuth2 API** (ID: JuO7zv2aoAdcklRn)

---

### 3. Process Sales Data

**ประเภท:** `n8n-nodes-base.code`  
**Version:** 2

#### หน้าที่
ประมวลผลข้อมูลยอดขายและคำนวณสถิติต่างๆ

#### โค้ดหลัก
```javascript
// ===================================================
// Process Sales Data (แก้ไข timestamp อย่างปลอดภัย)
// ===================================================
const salesData = [];

for (const item of $input.all()) {
  const json = item.json;

  if (json['วันที่ขาย'] && json['สินค้า'] && json['ประทับเวลา']) {
    // แยกวัน เดือน ปี และเวลา
    const timestampStr = json['ประทับเวลา'].trim();
    const [datePart, timePart] = timestampStr.split(',').map(s => s.trim());
    const [day, month, year] = datePart.split('/').map(Number);
    const [hour, minute, second] = timePart.split(':').map(Number);

    // สร้าง Date object อย่างปลอดภัย
    const isoTimestamp = new Date(year, month - 1, day, hour, minute, second).toISOString();

    salesData.push({
      date: json['วันที่ขาย'],                 
      product: json['สินค้า'],
      quantity: json['จำนวน'] || 0,
      revenue: json['ยอดขายรวม'] || 0,
      category: json['หมวดหมู่'] || 'N/A',
      timestamp: isoTimestamp
    });
  }
}
```

#### สถิติที่คำนวณ

1. **ยอดขายรวม (Total Revenue)**
```javascript
const totalRevenue = salesData.reduce((sum, item) => sum + item.revenue, 0);
```

2. **จำนวนสินค้ารวม (Total Quantity)**
```javascript
const totalQuantity = salesData.reduce((sum, item) => sum + item.quantity, 0);
```

3. **สถิติแต่ละสินค้า (Product Stats)**
```javascript
const productStats = {};
salesData.forEach(item => {
  if (!productStats[item.product]) {
    productStats[item.product] = { quantity: 0, revenue: 0 };
  }
  productStats[item.product].quantity += item.quantity;
  productStats[item.product].revenue += item.revenue;
});
```

4. **Top 10 สินค้าขายดี (Top Products)**
```javascript
const topProducts = Object.entries(productStats)
  .sort((a, b) => b[1].revenue - a[1].revenue)
  .slice(0, 10);
```

5. **ยอดขายรายวัน (Daily Sales)**
```javascript
const dailySales = {};
salesData.forEach(item => {
  if (!dailySales[item.date]) dailySales[item.date] = 0;
  dailySales[item.date] += item.revenue;
});
```

6. **สถิติตามหมวดหมู่ (Category Stats)**
```javascript
const categoryStats = {};
salesData.forEach(item => {
  if (!categoryStats[item.category]) {
    categoryStats[item.category] = 0;
  }
  categoryStats[item.category] += item.revenue;
});
```

#### Output Data Structure
```json
{
  "salesData": [...],
  "salesSummary": "รายการขาย 30 รายการแรก",
  "totalRevenue": 1250000,
  "totalQuantity": 150,
  "productStats": {...},
  "dataCount": 150,
  "analysisDate": "2025-10-28",
  "topProducts": [...],
  "dailySales": {...},
  "sortedDates": [...],
  "categoryStats": {...}
}
```

---

### 4. Generate Charts

**ประเภท:** `n8n-nodes-base.code`  
**Version:** 2

#### หน้าที่
สร้าง URL สำหรับกราฟ 3 แบบผ่าน QuickChart API

#### กราฟที่สร้าง

##### 1. 📊 Bar Chart: Top 10 สินค้าขายดี

**การตั้งค่า:**
```javascript
const barChartConfig = {
  type: 'bar',
  data: {
    labels: topProducts.map(([name]) => name),
    datasets: [{
      label: 'ยอดขาย (บาท)',
      data: topProducts.map(([, stats]) => stats.revenue),
      backgroundColor: 'rgba(54, 162, 235, 0.8)',
      borderColor: 'rgba(54, 162, 235, 1)',
      borderWidth: 2
    }]
  },
  options: {
    responsive: true,
    plugins: { 
      legend: { display: false }, 
      title: { 
        display: true, 
        text: '📊 Top 10 สินค้าขายดี', 
        font: {size: 20}, 
        color: '#333' 
      } 
    },
    scales: {
      y: { 
        beginAtZero: true, 
        ticks: {callback: v => v.toLocaleString('th-TH') + ' ฿'} 
      }
    }
  }
};
```

**ความหมาย:**
- แสดง 10 สินค้าที่มียอดขายสูงสุด
- แกน Y แสดงมูลค่าเป็นบาท
- สี: น้ำเงิน (Blue)

##### 2. 📈 Line Chart: แนวโน้มยอดขาย 14 วันล่าสุด

**การตั้งค่า:**
```javascript
const lastDates = data.sortedDates.slice(-14);
const lastSales = lastDates.map(d => data.dailySales[d] || 0);

const lineChartConfig = {
  type: 'line',
  data: { 
    labels: lastDates, 
    datasets: [{
      label: 'ยอดขายรายวัน (บาท)',
      data: lastSales,
      fill: true,
      backgroundColor: 'rgba(75,192,192,0.2)',
      borderColor: 'rgba(75,192,192,1)',
      borderWidth: 3,
      tension: 0.4,
      pointRadius: 5
    }]
  }
};
```

**ความหมาย:**
- แสดงยอดขายรายวัน 14 วันล่าสุด
- เส้นโค้ง (tension: 0.4) เพื่อความสวยงาม
- สี: เขียวมิ้นต์ (Mint Green)

##### 3. 🍰 Pie Chart: สัดส่วนยอดขายตามหมวดหมู่

**การตั้งค่า:**
```javascript
const pieChartLabels = Object.keys(data.categoryStats);
const pieChartData = Object.values(data.categoryStats);
const colors = [
  'rgba(255,99,132,0.8)',   // แดง
  'rgba(54,162,235,0.8)',   // น้ำเงิน
  'rgba(255,206,86,0.8)',   // เหลือง
  'rgba(75,192,192,0.8)',   // เขียวมิ้นต์
  'rgba(153,102,255,0.8)',  // ม่วง
  'rgba(255,159,64,0.8)',   // ส้ม
  // ... สีเพิ่มเติม
];

const pieChartConfig = {
  type: 'pie',
  data: {
    labels: pieChartLabels,
    datasets: [{
      data: pieChartData, 
      backgroundColor: colors.slice(0, pieChartLabels.length)
    }]
  }
};
```

**ความหมาย:**
- แสดงสัดส่วนยอดขายแต่ละหมวดหมู่
- ใช้สีที่แตกต่างกันสำหรับแต่ละหมวดหมู่

#### QuickChart API

**Base URL:**
```
https://quickchart.io/chart
```

**Parameters:**
- `width=800` - ความกว้าง 800px
- `height=500` - ความสูง 500px
- `backgroundColor=white` - พื้นหลังสีขาว
- `c={encoded_config}` - Config ที่ encode เป็น URL

**ตัวอย่าง URL:**
```
https://quickchart.io/chart?width=800&height=500&backgroundColor=white&c=%7B...%7D
```

#### Output
```json
{
  ...previousData,
  "barChartUrl": "https://quickchart.io/chart?...",
  "lineChartUrl": "https://quickchart.io/chart?...",
  "pieChartUrl": "https://quickchart.io/chart?...",
  "chartGenerated": true
}
```

---

### 5. Basic LLM Chain

**ประเภท:** `@n8n/n8n-nodes-langchain.chainLlm`  
**Version:** 1.7

#### หน้าที่
ใช้ AI (Google Gemini) วิเคราะห์ข้อมูลและสรุปเป็นรายงานภาษาไทย

#### Prompt Template

```
คุณเป็นนักวิเคราะห์ข้อมูลยอดขายมืออาชีพ วิเคราะห์ข้อมูลต่อไปนี้และสรุปเป็นภาษาไทยในรูปแบบรายงานที่กระชับและเข้าใจง่าย:

**📊 ข้อมูลยอดขาย ({{ dataCount }} รายการ)**
{{ salesSummary }}

**💰 สรุปยอดรวม:**
- ยอดขายรวม: {{ totalRevenue }} บาท
- จำนวนสินค้าที่ขาย: {{ totalQuantity }} ชิ้น
- จำนวนรายการทั้งหมด: {{ dataCount }} รายการ

**📦 สถิติแต่ละสินค้า:**
{{ productStats }}

กรุณาวิเคราะห์และสรุปใน 3 หัวข้อหลัก:

**1. 🏆 Top 3 สินค้าขายดี**
- เรียงตามยอดรายได้
- ระบุตัวเลขชัดเจน (จำนวนชิ้น + รายได้)
- บอกสัดส่วนเปอร์เซ็นต์ของยอดรวม

**2. 👀 โอกาสที่ควรติดตาม**
- สินค้าที่มีแนวโน้มดีหรือมีศักยภาพเติบโต
- แนะนำสินค้าที่ควรเพิ่ม stock หรือ promote
- จุดที่ควรระวังหรือปรับปรุง

**3. 📈 คาดการณ์เดือนหน้า**
- คาดการณ์แนวโน้มจากข้อมูลที่วิเคราะห์
- ข้อเสนอแนะเชิงกลยุทธ์
- เป้าหมายที่ควรตั้งไว้

ใช้ emoji ให้เหมาะสม เขียนกระชับ ตรงประเด็น เหมาะสำหรับอ่านบนมือถือ
ใช้ข้อความไม่เกิน 2500 ตัวอักษร
```

#### System Message
```
คุณเป็นผู้เชี่ยวชาญด้านการวิเคราะห์ยอดขายและการทำ business intelligence 
ที่สามารถให้คำแนะนำเชิงกลยุทธ์ได้อย่างชัดเจนและนำไปปฏิบัติได้จริง
```

#### ตัวอย่าง Output จาก AI
```
🏆 Top 3 สินค้าขายดี

1. iPhone 15 Pro Max - 45 ชิ้น, 675,000 บาท (35.5% ของยอดรวม)
2. MacBook Air M2 - 18 ชิ้น, 432,000 บาท (22.7% ของยอดรวม)
3. iPad Pro 11" - 32 ชิ้น, 384,000 บาท (20.2% ของยอดรวม)

👀 โอกาสที่ควรติดตาม

- AirPods Pro (Gen 2) มีแนวโน้มเติบโตต่อเนื่อง ควรเพิ่ม stock
- Apple Watch Series 9 ขายดีช่วงสุดสัปดาห์ ควร promote ในช่วงนี้
- Mac Mini ยอดขายต่ำ อาจต้องปรับกลยุทธ์ marketing

📈 คาดการณ์เดือนหน้า

คาดว่ายอดขายจะเพิ่มขึ้น 15-20% จากช่วงโปรโมชั่น
แนะนำเน้น iPhone และ MacBook ซึ่งเป็น high-value products
ตั้งเป้ายอดขาย 2.3 ล้านบาท
```

---

### 6. Google Gemini Chat Model

**ประเภท:** `@n8n/n8n-nodes-langchain.lmChatGoogleGemini`  
**Version:** 1

#### การตั้งค่า
```json
{
  "modelName": "models/gemini-2.5-pro"
}
```

#### Credentials ที่ต้องการ
- **Google Gemini (PaLM) API** (ID: FNTLr62FsyBeEYHP)

#### โมเดลที่แนะนำ
- `gemini-2.5-pro` - สมดุลระหว่างคุณภาพและความเร็ว (แนะนำ)
- `gemini-pro` - เร็วกว่า เหมาะกับงานที่ต้องการ response เร็ว
- `gemini-ultra` - คุณภาพสูงสุด แต่ช้ากว่า

---

### 7. Format LINE Message

**ประเภท:** `n8n-nodes-base.code`  
**Version:** 2

#### หน้าที่
จัดรูปแบบข้อความเพื่อส่งผ่าน LINE Messaging API

#### โค้ดตัวอย่าง (LINE Messaging API Format)
```javascript
const analysisText = $('Basic LLM Chain').item.json.text;
const data = $('Generate Charts1').item.json;

// จำกัดความยาวข้อความให้ไม่เกิน 5000 ตัวอักษร
const maxLength = 4800; // เผื่อไว้นิดหน่อย
let messageText = `📊 รายงานยอดขายสินค้า Apple
วันที่: ${data.analysisDate}

💰 สรุปยอดรวม
• ยอดขาย: ${data.totalRevenue.toLocaleString('th-TH')} บาท
• จำนวน: ${data.totalQuantity} ชิ้น
• รายการ: ${data.dataCount} รายการ

${analysisText}`;

// ตัดข้อความถ้ายาวเกิน
if (messageText.length > maxLength) {
  messageText = messageText.substring(0, maxLength) + '...';
}

// สร้าง messages array ตาม LINE Messaging API format
const messages = [
  {
    type: "text",
    text: messageText
  },
  {
    type: "image",
    originalContentUrl: data.barChartUrl,
    previewImageUrl: data.barChartUrl
  },
  {
    type: "image",
    originalContentUrl: data.lineChartUrl,
    previewImageUrl: data.lineChartUrl
  },
  {
    type: "image",
    originalContentUrl: data.pieChartUrl,
    previewImageUrl: data.pieChartUrl
  }
];

// สร้าง LINE Push Message payload
const lineMessage = {
  to: "USER_ID_หรือ_GROUP_ID",  // ⚠️ แก้ไขให้ตรงกับ User ID หรือ Group ID ของคุณ
  messages: messages
};

return [{
  json: {
    lineMessage: JSON.stringify(lineMessage)
  }
}];
```

#### วิธีหา User ID หรือ Group ID

**วิธีที่ 1: ใช้ Webhook (แนะนำ)**
1. ตั้งค่า Webhook URL ใน LINE Developers Console
2. เพิ่มเพื่อนกับบอท
3. ส่งข้อความไปที่บอท
4. Webhook จะได้รับ JSON ที่มี `userId` หรือ `groupId`

**วิธีที่ 2: ใช้เครื่องมือ**
- [LINE Bot Designer](https://developers.line.biz/console/) - ดูได้ใน webhook events
- ใช้ n8n webhook node เพื่อรับข้อมูลจาก LINE

**วิธีที่ 3: Hardcode (สำหรับทดสอบ)**
```javascript
const lineMessage = {
  to: "Uxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",  // User ID (ขึ้นต้นด้วย U)
  // หรือ
  to: "Cxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",  // Group ID (ขึ้นต้นด้วย C)
  messages: messages
};
```

---

### 8. Send via LINE Messaging API

**ประเภท:** `n8n-nodes-base.httpRequest`  
**Version:** 4.2

#### หน้าที่
ส่งข้อความและรูปภาพไปยัง LINE Official Account ผ่าน LINE Messaging API

#### การตั้งค่า (LINE Messaging API - Push Message)
```javascript
{
  "method": "POST",
  "url": "https://api.line.me/v2/bot/message/push",
  "authentication": "genericCredentialType",
  "genericAuthType": "httpHeaderAuth",
  "sendHeaders": true,
  "headerParameters": {
    "parameters": [
      {
        "name": "Authorization",
        "value": "Bearer YOUR_CHANNEL_ACCESS_TOKEN"
      },
      {
        "name": "Content-Type",
        "value": "application/json"
      }
    ]
  },
  "sendBody": true,
  "specifyBody": "json",
  "jsonBody": "={{ $json.lineMessage }}"
}
```

#### โครงสร้าง JSON Body
```json
{
  "to": "USER_ID หรือ GROUP_ID",
  "messages": [
    {
      "type": "text",
      "text": "ข้อความรายงาน"
    },
    {
      "type": "image",
      "originalContentUrl": "URL ของกราฟ Bar Chart",
      "previewImageUrl": "URL ของกราฟ Bar Chart"
    },
    {
      "type": "image",
      "originalContentUrl": "URL ของกราฟ Line Chart",
      "previewImageUrl": "URL ของกราฟ Line Chart"
    },
    {
      "type": "image",
      "originalContentUrl": "URL ของกราฟ Pie Chart",
      "previewImageUrl": "URL ของกราฟ Pie Chart"
    }
  ]
}
```

#### ตัวอย่างการสร้าง Message Object (ใน Format LINE Message node)
```javascript
const data = $('Generate Charts1').item.json;
const analysisText = $('Basic LLM Chain').item.json.text;

// สร้าง messages array
const messages = [
  {
    type: "text",
    text: `📊 รายงานยอดขายสินค้า Apple\nวันที่: ${data.analysisDate}\n\n💰 สรุปยอดรวม\n• ยอดขาย: ${data.totalRevenue.toLocaleString('th-TH')} บาท\n• จำนวน: ${data.totalQuantity} ชิ้น\n• รายการ: ${data.dataCount} รายการ\n\n${analysisText}`
  },
  {
    type: "image",
    originalContentUrl: data.barChartUrl,
    previewImageUrl: data.barChartUrl
  },
  {
    type: "image",
    originalContentUrl: data.lineChartUrl,
    previewImageUrl: data.lineChartUrl
  },
  {
    type: "image",
    originalContentUrl: data.pieChartUrl,
    previewImageUrl: data.pieChartUrl
  }
];

return [{
  json: {
    lineMessage: JSON.stringify({
      to: "USER_ID_หรือ_GROUP_ID",  // แก้ตามต้องการ
      messages: messages
    })
  }
}];
```

#### การส่งไปยัง LINE Group

**ขั้นตอน:**
1. เปิดใช้งาน "Allow bot to join group chats" ใน Channel settings
2. เพิ่มบอทเข้า LINE Group
3. หา Group ID:
   ```javascript
   // Webhook จะได้รับ event เมื่อบอทถูกเพิ่มเข้า group
   {
     "type": "join",
     "source": {
       "type": "group",
       "groupId": "Cxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
     }
   }
   ```
4. ใช้ Group ID ในการส่งข้อความ

**หมายเหตุ:**
- ส่งไปยัง Group ต้องใช้ `groupId` (ไม่ใช่ `userId`)
- บอทต้องอยู่ใน Group นั้นๆ
- สามารถส่งไปหลาย Group พร้อมกันโดยเรียก API หลายครั้ง

#### Message Types ที่รองรับ

LINE Messaging API รองรับหลาย message types:

1. **Text Message**
```json
{
  "type": "text",
  "text": "ข้อความ"
}
```

2. **Image Message**
```json
{
  "type": "image",
  "originalContentUrl": "https://...",
  "previewImageUrl": "https://..."
}
```

3. **Flex Message** (แนะนำสำหรับรายงานที่สวยงาม)
```json
{
  "type": "flex",
  "altText": "รายงานยอดขาย",
  "contents": {
    "type": "bubble",
    "body": {
      "type": "box",
      "layout": "vertical",
      "contents": [...]
    }
  }
}
```

4. **Template Message** (Buttons, Confirm, Carousel)
```json
{
  "type": "template",
  "altText": "รายงาน",
  "template": {
    "type": "buttons",
    "text": "ดูรายงานเพิ่มเติม",
    "actions": [...]
  }
}
```

#### การส่งหลายข้อความ
LINE Messaging API อนุญาตให้ส่งได้สูงสุด **5 messages** ต่อ 1 request

---

### 💎 การปรับแต่ง LINE Message ให้สวยงามด้วย Flex Message

หากต้องการรายงานที่สวยงามและ interactive มากขึ้น สามารถใช้ **Flex Message** แทน Text Message:

```javascript
const data = $('Generate Charts1').item.json;
const analysisText = $('Basic LLM Chain').item.json.text;

// สร้าง Flex Message
const flexMessage = {
  type: "flex",
  altText: `รายงานยอดขาย Apple - ${data.analysisDate}`,
  contents: {
    type: "bubble",
    size: "mega",
    header: {
      type: "box",
      layout: "vertical",
      contents: [
        {
          type: "text",
          text: "📊 รายงานยอดขาย",
          weight: "bold",
          size: "xl",
          color: "#ffffff"
        },
        {
          type: "text",
          text: data.analysisDate,
          size: "sm",
          color: "#ffffff"
        }
      ],
      backgroundColor: "#667eea",
      paddingAll: "20px"
    },
    body: {
      type: "box",
      layout: "vertical",
      contents: [
        {
          type: "box",
          layout: "vertical",
          margin: "lg",
          spacing: "sm",
          contents: [
            {
              type: "box",
              layout: "baseline",
              spacing: "sm",
              contents: [
                {
                  type: "text",
                  text: "ยอดขายรวม",
                  color: "#aaaaaa",
                  size: "sm",
                  flex: 2
                },
                {
                  type: "text",
                  text: `${data.totalRevenue.toLocaleString('th-TH')} ฿`,
                  wrap: true,
                  color: "#666666",
                  size: "md",
                  weight: "bold",
                  flex: 3,
                  align: "end"
                }
              ]
            },
            {
              type: "box",
              layout: "baseline",
              spacing: "sm",
              contents: [
                {
                  type: "text",
                  text: "จำนวนสินค้า",
                  color: "#aaaaaa",
                  size: "sm",
                  flex: 2
                },
                {
                  type: "text",
                  text: `${data.totalQuantity} ชิ้น`,
                  wrap: true,
                  color: "#666666",
                  size: "md",
                  flex: 3,
                  align: "end"
                }
              ]
            }
          ]
        },
        {
          type: "separator",
          margin: "lg"
        },
        {
          type: "box",
          layout: "vertical",
          margin: "lg",
          contents: [
            {
              type: "text",
              text: "🏆 Top 3 สินค้า",
              size: "md",
              weight: "bold",
              margin: "md"
            },
            ...data.topProducts.slice(0, 3).map(([product, stats], index) => ({
              type: "box",
              layout: "baseline",
              margin: "md",
              contents: [
                {
                  type: "text",
                  text: `${index + 1}. ${product}`,
                  size: "sm",
                  color: "#555555",
                  flex: 3,
                  wrap: true
                },
                {
                  type: "text",
                  text: `${stats.revenue.toLocaleString('th-TH')} ฿`,
                  size: "sm",
                  color: "#111111",
                  align: "end",
                  flex: 2
                }
              ]
            }))
          ]
        }
      ]
    },
    footer: {
      type: "box",
      layout: "vertical",
      spacing: "sm",
      contents: [
        {
          type: "button",
          style: "primary",
          height: "sm",
          action: {
            type: "uri",
            label: "ดูกราฟทั้งหมด",
            uri: "https://your-dashboard-url.com"  // แก้ตาม URL จริง
          },
          color: "#667eea"
        }
      ],
      flex: 0
    }
  }
};

// สร้าง messages array
const messages = [
  flexMessage,
  {
    type: "image",
    originalContentUrl: data.barChartUrl,
    previewImageUrl: data.barChartUrl
  },
  {
    type: "image",
    originalContentUrl: data.lineChartUrl,
    previewImageUrl: data.lineChartUrl
  },
  {
    type: "image",
    originalContentUrl: data.pieChartUrl,
    previewImageUrl: data.pieChartUrl
  }
];

return [{
  json: {
    lineMessage: JSON.stringify({
      to: "USER_ID_หรือ_GROUP_ID",
      messages: messages
    })
  }
}];
```

**เครื่องมือสร้าง Flex Message:**
- [Flex Message Simulator](https://developers.line.biz/flex-simulator/) - ทดสอบและ preview Flex Message

**ลำดับการส่ง:**
1. ข้อความรายงาน (Text Message)
2. กราฟ Bar Chart (Image Message)
3. กราฟ Line Chart (Image Message)
4. กราฟ Pie Chart (Image Message)

#### Credentials ที่ต้องการ
- **Channel Access Token** จาก LINE Developers Console
- **User ID** หรือ **Group ID** ของผู้รับ

#### ข้อจำกัด
- Text message: สูงสุด 5,000 ตัวอักษร
- Push message limit: ขึ้นอยู่กับ plan ของ LINE Official Account
- Image URL ต้อง HTTPS และเข้าถึงได้สาธารณะ

---

### 9. Format Email Message

**ประเภท:** `n8n-nodes-base.code`  
**Version:** 2

#### หน้าที่
จัดรูปแบบอีเมลแบบ HTML

#### โค้ดตัวอย่าง
```javascript
const analysisText = $('Basic LLM Chain').item.json.text;
const data = $('Generate Charts1').item.json;

// แปลง line breaks เป็น HTML
const formattedAnalysis = analysisText.replace(/\n/g, '<br>');

const htmlBody = `
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <style>
    body { 
      font-family: 'Sarabun', Arial, sans-serif; 
      line-height: 1.6; 
      color: #333;
      max-width: 800px;
      margin: 0 auto;
      padding: 20px;
    }
    .header { 
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      color: white;
      padding: 30px;
      border-radius: 10px;
      text-align: center;
      margin-bottom: 30px;
    }
    .header h1 { margin: 0; font-size: 28px; }
    .summary { 
      background: #f7f9fc;
      padding: 20px;
      border-radius: 8px;
      border-left: 4px solid #667eea;
      margin-bottom: 30px;
    }
    .summary-item { 
      display: flex;
      justify-content: space-between;
      padding: 10px 0;
      border-bottom: 1px solid #e0e0e0;
    }
    .summary-item:last-child { border-bottom: none; }
    .summary-label { font-weight: 600; color: #666; }
    .summary-value { font-weight: 700; color: #667eea; font-size: 18px; }
    .analysis { 
      background: white;
      padding: 25px;
      border-radius: 8px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
      margin-bottom: 30px;
    }
    .chart { 
      margin: 20px 0;
      text-align: center;
    }
    .chart img { 
      max-width: 100%;
      border-radius: 8px;
      box-shadow: 0 4px 6px rgba(0,0,0,0.1);
    }
    .chart-title {
      font-size: 18px;
      font-weight: 600;
      color: #333;
      margin-bottom: 15px;
    }
    .footer {
      text-align: center;
      color: #999;
      font-size: 12px;
      margin-top: 40px;
      padding-top: 20px;
      border-top: 1px solid #e0e0e0;
    }
  </style>
</head>
<body>
  <div class="header">
    <h1>📊 รายงานยอดขายสินค้า Apple</h1>
    <p>วันที่: ${data.analysisDate}</p>
  </div>

  <div class="summary">
    <h2>💰 สรุปยอดรวม</h2>
    <div class="summary-item">
      <span class="summary-label">ยอดขายรวม:</span>
      <span class="summary-value">${data.totalRevenue.toLocaleString('th-TH')} บาท</span>
    </div>
    <div class="summary-item">
      <span class="summary-label">จำนวนสินค้า:</span>
      <span class="summary-value">${data.totalQuantity} ชิ้น</span>
    </div>
    <div class="summary-item">
      <span class="summary-label">จำนวนรายการ:</span>
      <span class="summary-value">${data.dataCount} รายการ</span>
    </div>
  </div>

  <div class="analysis">
    <h2>🤖 วิเคราะห์โดย AI</h2>
    ${formattedAnalysis}
  </div>

  <div class="chart">
    <div class="chart-title">📊 Top 10 สินค้าขายดี</div>
    <img src="${data.barChartUrl}" alt="Bar Chart">
  </div>

  <div class="chart">
    <div class="chart-title">📈 แนวโน้มยอดขาย 14 วันล่าสุด</div>
    <img src="${data.lineChartUrl}" alt="Line Chart">
  </div>

  <div class="chart">
    <div class="chart-title">🍰 สัดส่วนยอดขายตามหมวดหมู่</div>
    <img src="${data.pieChartUrl}" alt="Pie Chart">
  </div>

  <div class="footer">
    <p>รายงานนี้สร้างโดยระบบอัตโนมัติ | Powered by n8n & Google Gemini</p>
  </div>
</body>
</html>
`;

return [{
  json: {
    subject: `📊 รายงานยอดขาย Apple - ${data.analysisDate}`,
    htmlBody,
    to: 'recipient@example.com'  // แก้ไขตามต้องการ
  }
}];
```

#### คุณสมบัติของอีเมล
- **Responsive Design** - ดูได้ทั้งมือถือและคอมพิวเตอร์
- **สวยงาม** - ใช้ Gradient, Shadow, และ Modern UI
- **รูปภาพกราฟฝังใน Email** - ไม่ต้องดาวน์โหลดแยก
- **รองรับภาษาไทย** - ใช้ font Sarabun

---

### 10. Send a message (Gmail)

**ประเภท:** `n8n-nodes-base.gmail`  
**Version:** 2.2

#### หน้าที่
ส่งอีเมลผ่าน Gmail API

#### การตั้งค่า
```json
{
  "sendTo": "={{ $json.to }}",
  "subject": "={{ $json.subject }}",
  "emailFormat": "html",
  "message": "={{ $json.htmlBody }}"
}
```

#### Credentials ที่ต้องการ
- **Gmail OAuth2** (ID: HbRQKRDJEPtML18H)

#### การตั้งค่า Gmail OAuth2
1. สร้าง Project ใน Google Cloud Console
2. เปิดใช้งาน Gmail API
3. สร้าง OAuth 2.0 Client ID
4. เพิ่ม Scopes:
   - `https://www.googleapis.com/auth/gmail.send`
   - `https://www.googleapis.com/auth/gmail.compose`

---

## ⚙️ การตั้งค่าและ Prerequisites

### 📌 LINE Notify vs LINE Messaging API

Workflow นี้ใช้ **LINE Messaging API** (ไม่ใช่ LINE Notify) เนื่องจากต้องการส่งข้อความแบบ rich content พร้อมรูปภาพหลายรูป

| Feature | LINE Notify | LINE Messaging API |
|---------|-------------|-------------------|
| **การตั้งค่า** | ง่าย (แค่ generate token) | ซับซ้อนกว่า (ต้องสร้าง channel) |
| **รูปแบบข้อความ** | Text + 1 รูป | Text, Image, Flex, Template, Video, etc. |
| **จำนวนรูปภาพ** | 1 รูปต่อข้อความ | 5 messages ต่อ request |
| **Interactive** | ไม่มี | มี (buttons, carousel, flex) |
| **Quota** | ไม่จำกัด | 500 push/month (Free) |
| **Use Case** | แจ้งเตือนง่ายๆ | Rich content, interactive bot |
| **ราคา** | ฟรี | Free/Paid plans |

**เหตุผลที่เลือก LINE Messaging API:**
- ✅ ส่งกราฟได้ 3 รูปพร้อมกัน
- ✅ ใช้ Flex Message ทำรายงานสวยงาม
- ✅ สามารถทำ interactive bot ได้ในอนาคต
- ✅ Professional มากกว่า

---

### 1. ความต้องการของระบบ

#### Software
- **n8n** (version ล่าสุด)
- **Node.js** 18.x หรือสูงกว่า
- **Internet Connection** สำหรับ API calls

#### Accounts ที่ต้องการ
1. **Google Account** สำหรับ:
   - Google Sheets
   - Gmail
   - Google Gemini API
2. **LINE Account** สำหรับ:
   - LINE Notify หรือ LINE Messaging API

### 2. API Keys และ Credentials

#### Google Gemini API
1. ไปที่ [Google AI Studio](https://makersuite.google.com/app/apikey)
2. คลิก "Get API Key"
3. Copy API key มาใช้

#### Google Sheets OAuth2
1. ไปที่ [Google Cloud Console](https://console.cloud.google.com/)
2. สร้าง Project ใหม่
3. เปิดใช้งาน Google Sheets API
4. สร้าง OAuth 2.0 Client ID
5. เพิ่ม Authorized redirect URIs: `https://YOUR_N8N_URL/rest/oauth2-credential/callback`
6. Download JSON credentials

#### Gmail OAuth2
1. ใช้ Project เดียวกับ Google Sheets
2. เปิดใช้งาน Gmail API
3. ใช้ OAuth Client เดียวกัน หรือสร้างใหม่

#### LINE Messaging API
1. ไปที่ [LINE Developers Console](https://developers.line.biz/console/)
2. สร้าง Provider (ถ้ายังไม่มี)
3. สร้าง Messaging API Channel
4. ในหน้า Channel settings:
   - Copy **Channel Access Token** (Long-lived)
   - เปิดใช้งาน "Use webhooks" (ถ้าต้องการ)
   - เปิดใช้งาน "Allow bot to join group chats" (ถ้าส่งไป group)
5. หา **User ID** หรือ **Group ID**:
   - เพิ่มเพื่อนกับ Official Account
   - ส่งข้อความไปที่บอท
   - ใช้ Webhook หรือ [LINE Bot Designer](https://developers.line.biz/console/) ดู User ID

### 3. โครงสร้าง Google Sheets

สร้าง Sheet ด้วยคอลัมน์ดังนี้:

| ประทับเวลา | วันที่ขาย | สินค้า | จำนวน | ยอดขายรวม | หมวดหมู่ |
|---|---|---|---|---|---|
| 26/10/2025, 21:46:47 | 26/10/2025 | iPhone 15 Pro Max | 2 | 79,800 | iPhone |
| 26/10/2025, 14:30:15 | 26/10/2025 | MacBook Air M2 | 1 | 39,900 | Mac |
| 25/10/2025, 16:22:33 | 25/10/2025 | iPad Pro 11" | 1 | 29,900 | iPad |

**Tips:**
- ใช้ Google Forms เชื่อมกับ Sheet เพื่อบันทึกข้อมูล
- ตั้งค่า Form ให้บันทึก timestamp อัตโนมัติ

---

## 🚀 วิธีการติดตั้งและใช้งาน

### ขั้นตอนที่ 1: Import Workflow

1. เปิด n8n
2. คลิก "Import workflow"
3. Paste JSON code หรืออัปโหลดไฟล์
4. คลิก "Import"

### ขั้นตอนที่ 2: ตั้งค่า Credentials

#### Google Sheets
1. คลิกที่ node "Get row(s) in sheet"
2. คลิก "Create New Credential"
3. เลือก "OAuth2"
4. กรอก Client ID และ Client Secret
5. คลิก "Connect my account"
6. Authorize access

#### Google Gemini
1. คลิกที่ node "Google Gemini Chat Model"
2. คลิก "Create New Credential"
3. กรอก API Key
4. บันทึก

#### Gmail
1. คลิกที่ node "Send a message"
2. คลิก "Create New Credential"
3. เลือก "OAuth2"
4. Follow the same steps as Google Sheets

#### LINE Messaging API
1. คลิกที่ node "Send via LINE Messaging API1"
2. ตั้งค่า Authentication:
   - Method: Generic Credential Type
   - Credential Type: Header Auth
3. ใส่ Headers:
   - Name: `Authorization`
   - Value: `Bearer YOUR_CHANNEL_ACCESS_TOKEN`
   - Name: `Content-Type`
   - Value: `application/json`
4. ตั้งค่า Body:
   - Body Content Type: JSON
   - JSON: ใส่โครงสร้างตามที่กำหนดใน node "Format LINE Message"

### ขั้นตอนที่ 3: แก้ไข Document ID

1. คลิกที่ node "Get row(s) in sheet"
2. แก้ไข `documentId` เป็น ID ของ Google Sheets ของคุณ
3. เลือก Sheet name ที่ถูกต้อง

### ขั้นตอนที่ 4: แก้ไขอีเมลผู้รับ

ใน node "Format Email Message":
```javascript
to: 'your-email@example.com'  // แก้ตรงนี้
```

### ขั้นตอนที่ 5: ทดสอบ Workflow

1. คลิก "Execute Workflow" ที่มุมล่างขวา
2. ดูผลลัพธ์แต่ละ node
3. ตรวจสอบว่าได้รับอีเมลและข้อความ LINE

### ขั้นตอนที่ 6: เปิดใช้งาน

1. คลิก "Active" เพื่อเปิดใช้งาน Schedule
2. Workflow จะรันอัตโนมัติทุกวันเวลา 08:00 น.

---

## 🔧 การปรับแต่ง

### เปลี่ยนเวลารัน

แก้ไข Cron Expression ใน node "Schedule Trigger":

```javascript
// ทุกวันเวลา 09:00
"0 9 * * *"

// ทุกวันจันทร์เวลา 08:00
"0 8 * * 1"

// วันที่ 1 ของทุกเดือนเวลา 08:00
"0 8 1 * *"

// ทุก 2 ชั่วโมง
"0 */2 * * *"
```

### เปลี่ยนจำนวนสินค้าใน Top Products

ใน node "Process Sales Data":
```javascript
// แก้จาก 10 เป็น 5
.slice(0, 5)
```

### เปลี่ยนจำนวนวันใน Line Chart

ใน node "Generate Charts":
```javascript
// แก้จาก 14 เป็น 30 วัน
const lastDates = data.sortedDates.slice(-30);
```

### เพิ่มผู้รับอีเมล

ใน node "Send a message":
```javascript
{
  "sendTo": "person1@example.com,person2@example.com,person3@example.com"
}
```

### เปลี่ยน Theme สีกราฟ

ใน node "Generate Charts":
```javascript
// สีแดง
backgroundColor: 'rgba(255, 99, 132, 0.8)',
borderColor: 'rgba(255, 99, 132, 1)',

// สีเขียว
backgroundColor: 'rgba(75, 192, 192, 0.8)',
borderColor: 'rgba(75, 192, 192, 1)',

// สีส้ม
backgroundColor: 'rgba(255, 159, 64, 0.8)',
borderColor: 'rgba(255, 159, 64, 1)',
```

### เพิ่ม Error Handling สำหรับ LINE API

เพื่อให้ workflow ทำงานได้แม้ LINE API มีปัญหา สามารถเพิ่ม Error Handler ใน node "Send via LINE Messaging API":

```javascript
// ใน node "Format LINE Message" - เพิ่ม error handling
try {
  const data = $('Generate Charts1').item.json;
  const analysisText = $('Basic LLM Chain').item.json.text;
  
  // Validate data
  if (!data.barChartUrl || !data.lineChartUrl || !data.pieChartUrl) {
    throw new Error('กราฟไม่ครบ');
  }
  
  // จำกัดความยาว
  let messageText = `📊 รายงานยอดขาย...[truncated]`;
  if (messageText.length > 5000) {
    messageText = messageText.substring(0, 4997) + '...';
  }
  
  const messages = [...]; // messages array
  
  return [{
    json: {
      lineMessage: JSON.stringify({
        to: process.env.LINE_USER_ID || "USER_ID",
        messages: messages
      }),
      success: true
    }
  }];
  
} catch (error) {
  // Return error message แต่ไม่ fail workflow
  return [{
    json: {
      lineMessage: JSON.stringify({
        to: process.env.LINE_USER_ID || "USER_ID",
        messages: [{
          type: "text",
          text: `⚠️ เกิดข้อผิดพลาดในการสร้างรายงาน: ${error.message}`
        }]
      }),
      success: false,
      error: error.message
    }
  }];
}
```

**เพิ่ม Retry Logic:**
```javascript
// ใน node หลัง LINE API - ตรวจสอบ response
const response = $input.first().json;

if (response.statusCode === 429) {
  // Rate limited - รอแล้วลองใหม่
  throw new Error('Rate limited - จะลองใหม่อีกครั้ง');
} else if (response.statusCode >= 500) {
  // Server error - ลองใหม่
  throw new Error('LINE API server error');
} else if (response.statusCode === 403) {
  // Bot ไม่ได้เป็นเพื่อน - แจ้งเตือน
  console.log('User ต้องเพิ่มบอทเป็นเพื่อนก่อน');
}
```

**ใช้ n8n Error Workflow:**
สามารถตั้งค่า Error Workflow แยกใน n8n เพื่อจัดการ errors:
1. ไปที่ Workflow Settings
2. เปิดใช้ Error Workflow
3. เลือก workflow สำหรับจัดการ errors
4. จะถูกเรียกทุกครั้งที่เกิด error

---

## 🐛 การแก้ไขปัญหา

### ปัญหา: Workflow ไม่รัน

**สาเหตุที่เป็นไปได้:**
1. Workflow ไม่ได้เปิดใช้งาน (Active)
2. Cron Expression ผิด
3. n8n ไม่ได้ทำงาน

**วิธีแก้:**
```bash
# ตรวจสอบสถานะ n8n
pm2 status n8n

# Restart n8n
pm2 restart n8n

# ดู logs
pm2 logs n8n
```

### ปัญหา: Google Sheets ดึงข้อมูลไม่ได้

**สาเหตุที่เป็นไปได้:**
1. Document ID ผิด
2. Sheet name ผิด
3. OAuth token หมดอายุ

**วิธีแก้:**
1. คลิกที่ node "Get row(s) in sheet"
2. คลิก "Reconnect account"
3. Authorize ใหม่

### ปัญหา: AI ไม่วิเคราะห์ข้อมูล

**สาเหตุที่เป็นไปได้:**
1. Google Gemini API key ผิดหรือหมดอายุ
2. Quota หมด
3. Prompt ยาวเกินไป

**วิธีแก้:**
1. ตรวจสอบ API key
2. ดู [Gemini API Console](https://makersuite.google.com/) เพื่อดู quota
3. ลดจำนวนข้อมูลใน prompt

### ปัญหา: กราฟไม่แสดง

**สาเหตุที่เป็นไปได้:**
1. QuickChart API มีปัญหา
2. Config ผิด
3. ข้อมูลไม่ครบ

**วิธีแก้:**
1. ลอง URL กราฟใน browser
2. ตรวจสอบ error ใน node "Generate Charts"
3. ดูว่าข้อมูลจาก node ก่อนหน้าครบหรือไม่

### ปัญหา: อีเมลส่งไม่ออก

**สาเหตุที่เป็นไปได้:**
1. Gmail OAuth token หมดอายุ
2. อีเมลผู้รับผิด
3. HTML มี error

**วิธีแก้:**
1. Reconnect Gmail account
2. ตรวจสอบรูปแบบอีเมล
3. ลองส่งอีเมลทดสอบ

### ปัญหา: LINE ส่งข้อความไม่ได้

**สาเหตุที่เป็นไปได้:**
1. Channel Access Token ผิดหรือหมดอายุ
2. User ID หรือ Group ID ผิด
3. ข้อความยาวเกิน 5,000 ตัวอักษร
4. Image URL ไม่สามารถเข้าถึงได้ (ต้อง HTTPS)
5. เกิน Push message quota

**วิธีแก้:**

**1. ตรวจสอบ Token:**
```bash
# ทดสอบ Token ด้วย curl
curl -X POST https://api.line.me/v2/bot/message/push \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer YOUR_CHANNEL_ACCESS_TOKEN' \
  -d '{
    "to": "USER_ID",
    "messages": [{"type": "text", "text": "test"}]
  }'
```

**2. ตรวจสอบ User ID/Group ID:**
- User ID ขึ้นต้นด้วย `U` (เช่น `Uxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`)
- Group ID ขึ้นต้นด้วย `C` (เช่น `Cxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`)
- Room ID ขึ้นต้นด้วย `R` (เช่น `Rxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`)

**3. ตรวจสอบความยาวข้อความ:**
```javascript
// เพิ่มการตรวจสอบใน code
if (messageText.length > 5000) {
  messageText = messageText.substring(0, 4997) + '...';
}
```

**4. ตรวจสอบ Image URLs:**
- ต้องเป็น HTTPS (ไม่ใช่ HTTP)
- ขนาดไฟล์ไม่เกิน 10 MB
- Format: JPEG หรือ PNG
- QuickChart URLs ควรใช้งานได้ปกติ

**5. ตรวจสอบ Quota:**
- ไปที่ [LINE Developers Console](https://developers.line.biz/console/)
- เช็ค Push message usage
- Free plan: 500 messages/month
- ถ้าหมดต้อง upgrade plan

**6. ดู Error Response:**
ใน n8n execution log จะมี error message จาก LINE API:
```json
{
  "message": "The request body has 2 error(s)",
  "details": [
    {
      "message": "May not be empty",
      "property": "messages[0].text"
    }
  ]
}
```

**Common Error Codes:**
- `400` - Invalid request (ตรวจสอบ JSON format)
- `401` - Invalid access token
- `403` - Bot ไม่สามารถส่งถึง user นี้ได้ (user ต้องเป็นเพื่อนกับบอทก่อน)
- `404` - Invalid userId/groupId
- `429` - Rate limit exceeded

### ปัญหา: Timestamp แสดงผลผิด

**สาเหตุ:**
Timezone ไม่ตรง

**วิธีแก้:**
```javascript
// เพิ่ม timezone offset
const date = new Date(year, month - 1, day, hour, minute, second);
// แปลงเป็น GMT+7 (Bangkok)
date.setHours(date.getHours() + 7);
```

---

## 📊 ตัวอย่างผลลัพธ์

### ตัวอย่างอีเมลที่ได้รับ

**Subject:** 📊 รายงานยอดขาย Apple - 2025-10-28

**Body:**
```
📊 รายงานยอดขายสินค้า Apple
วันที่: 2025-10-28

💰 สรุปยอดรวม
• ยอดขายรวม: 1,905,000 บาท
• จำนวนสินค้า: 127 ชิ้น
• จำนวนรายการ: 127 รายการ

🤖 วิเคราะห์โดย AI

🏆 Top 3 สินค้าขายดี

1. iPhone 15 Pro Max (256GB) - 28 ชิ้น, 1,344,000 บาท (70.5%)
   • สินค้าขายดีที่สุด! ควรรักษา stock ให้พร้อม
   
2. MacBook Air M2 (16GB) - 12 ชิ้น, 478,800 บาท (25.1%)
   • ความต้องการเพิ่มขึ้นในช่วงสัปดาห์นี้
   
3. AirPods Pro (Gen 2) - 15 ชิ้น, 82,500 บาท (4.3%)
   • สินค้าเสริมขายดี มักซื้อพร้อม iPhone

👀 โอกาสที่ควรติดตาม

• iPad Pro ขายได้น้อย อาจเนื่องจากราคาสูง แนะนำทำโปรโมชั่น
• Apple Watch Series 9 มีแนวโน้มดีในช่วงสุดสัปดาห์
• Mac Mini มีศักยภาพ แต่ต้อง education marketing

📈 คาดการณ์เดือนหน้า

จากแนวโน้ม 14 วันล่าสุด คาดว่า:
• ยอดขายจะเติบโต 18-22% จากช่วงโปรโมชั่นสิ้นเดือน
• iPhone จะยังคงนำทีม แต่ MacBook จะเพิ่มสัดส่วน
• ตั้งเป้า 2.2 ล้านบาทสำหรับเดือนถัดไป

[กราฟแสดงผลอยู่ด้านล่าง]
```

### ตัวอย่างข้อความ LINE

**Message 1: Text Message**
```
📊 รายงานยอดขาย Apple
วันที่: 28/10/2025

💰 สรุปยอดรวม
• ยอดขาย: 1,905,000 บาท
• จำนวน: 127 ชิ้น
• รายการ: 127 รายการ

🏆 Top 3 สินค้าขายดี

1. iPhone 15 Pro Max (256GB) - 28 ชิ้น, 1,344,000 บาท (70.5%)
   • สินค้าขายดีที่สุด! ควรรักษา stock ให้พร้อม

2. MacBook Air M2 (16GB) - 12 ชิ้น, 478,800 บาท (25.1%)
   • ความต้องการเพิ่มขึ้นในช่วงสัปดาห์นี้

3. AirPods Pro (Gen 2) - 15 ชิ้น, 82,500 บาท (4.3%)
   • สินค้าเสริมขายดี มักซื้อพร้อม iPhone

👀 โอกาสที่ควรติดตาม

• iPad Pro ขายได้น้อย อาจเนื่องจากราคาสูง แนะนำทำโปรโมชั่น
• Apple Watch Series 9 มีแนวโน้มดีในช่วงสุดสัปดาห์
• Mac Mini มีศักยภาพ แต่ต้อง education marketing

📈 คาดการณ์เดือนหน้า

คาดว่ายอดขายจะเติบโต 18-22%
ตั้งเป้า 2.2 ล้านบาทสำหรับเดือนถัดไป
```

**Message 2-4: Image Messages**
- รูปกราฟ Bar Chart (Top 10 สินค้าขายดี)
- รูปกราฟ Line Chart (แนวโน้มยอดขาย 14 วัน)
- รูปกราฟ Pie Chart (สัดส่วนตามหมวดหมู่)

**หมายเหตุ:**
- ข้อความและรูปภาพจะแสดงตามลำดับ
- รูปภาพสามารถแตะเพื่อดูขนาดเต็ม
- กราฟจาก QuickChart จะ render สวยงามและชัดเจน

---

## 📈 การวัดผลและ Metrics

### KPIs ที่ติดตาม

1. **ยอดขายรวม (Total Revenue)**
   - เป้าหมาย: 2,000,000 บาท/เดือน
   - Growth: 15-20% ต่อเดือน

2. **จำนวนรายการ (Transaction Count)**
   - เป้าหมาย: 150 รายการ/เดือน
   - Average: 5 รายการ/วัน

3. **มูลค่าเฉลี่ย (Average Order Value)**
   - คำนวณ: Total Revenue / Transaction Count
   - เป้าหมาย: 15,000 บาท/รายการ

4. **สินค้าขายดี (Best Sellers)**
   - Top 10 products
   - % ของยอดรวม

5. **Category Performance**
   - iPhone, iPad, Mac, Accessories
   - สัดส่วนแต่ละหมวดหมู่

### การปรับปรุงต่อเนื่อง

**รายสัปดาห์:**
- ตรวจสอบความถูกต้องของข้อมูล
- ปรับแต่ง prompt ของ AI
- เพิ่ม/ลด สินค้าตาม performance

**รายเดือน:**
- วิเคราะห์ trend ระยะยาว
- ปรับ target ตาม seasonality
- Update AI model หากมี version ใหม่

**รายไตรมาส:**
- Review ทั้งระบบ
- เพิ่มฟีเจอร์ใหม่
- Optimize performance

---

## 🎯 Best Practices

### 1. การจัดการข้อมูล

**DO:**
- ✅ ตรวจสอบข้อมูลก่อนบันทึกใน Sheet
- ✅ ใช้ Data Validation ใน Google Sheets
- ✅ Backup ข้อมูลเป็นประจำ
- ✅ ใช้ timestamp ที่ชัดเจน

**DON'T:**
- ❌ ลบข้อมูลเก่าโดยไม่ backup
- ❌ เปลี่ยน schema โดยไม่แก้ไข workflow
- ❌ ใช้ตัวอักษรพิเศษใน column names

### 2. Security

**DO:**
- ✅ เก็บ API keys และ Channel Access Token ใน environment variables
- ✅ ใช้ OAuth แทน API keys เมื่อเป็นไปได้
- ✅ จำกัดสิทธิ์ access ของ API
- ✅ Rotate tokens เป็นระยะ
- ✅ เปิดใช้ Webhook signature validation สำหรับ LINE (ถ้ามี webhook)
- ✅ ใช้ Channel Secret เพื่อ verify requests

**DON'T:**
- ❌ Share credentials ใน public repositories
- ❌ ใช้ hardcoded passwords หรือ tokens
- ❌ ให้สิทธิ์เกินจำเป็น
- ❌ เก็บ Channel Access Token ใน client-side code
- ❌ แชร์ User ID หรือ Group ID ต่อสาธารณะ

**การรักษาความปลอดภัย Channel Access Token:**
```javascript
// ❌ ไม่ดี - Hardcode token
const token = "Bearer abc123xyz...";

// ✅ ดี - ใช้ environment variable
const token = `Bearer ${process.env.LINE_CHANNEL_ACCESS_TOKEN}`;

// ✅ ดีที่สุด - ใช้ n8n credentials
// ตั้งค่าใน n8n credentials manager
```

**การตรวจสอบ Webhook Signature (ถ้ามี):**
```javascript
const crypto = require('crypto');

function validateSignature(body, signature, channelSecret) {
  const hash = crypto
    .createHmac('SHA256', channelSecret)
    .update(JSON.stringify(body))
    .digest('base64');
  return hash === signature;
}
```

### 3. Performance

**DO:**
- ✅ จำกัดจำนวนข้อมูลที่ process ครั้งละไม่เกิน 1,000 รายการ
- ✅ ใช้ batch processing สำหรับข้อมูลมาก
- ✅ Cache ผลลัพธ์เมื่อเป็นไปได้
- ✅ Monitor execution time

**DON'T:**
- ❌ Process ข้อมูลทั้งหมดในครั้งเดียวถ้ามี 10,000+ รายการ
- ❌ สร้างกราฟที่มีจุดข้อมูลมากเกินไป (> 100 points)
- ❌ ใช้ AI สำหรับทุกรายการ (ใช้กับ summary เท่านั้น)

### 4. Monitoring

**ควรตรวจสอบ:**
- ⏱️ Execution time แต่ละ node
- ❗ Error rate
- 📊 Data quality
- 💰 API usage และ costs
- 📱 LINE Push message quota (500/month สำหรับ Free plan)

**ตั้ง Alerts สำหรับ:**
- Workflow ล้มเหลว 3 ครั้งติดต่อกัน
- Execution time เกิน 5 นาที
- ข้อมูลผิดปกติ (เช่น ยอดขาย = 0)
- LINE quota ใกล้หมด (เหลือ < 50 messages)
- HTTP 429 (Rate limit exceeded) จาก APIs

**LINE API Rate Limits:**
- **Push Message:** 500 messages/month (Free), unlimited (Paid)
- **Reply Message:** ไม่จำกัด (ใช้ได้เมื่อมี incoming message)
- **Multicast:** 500 recipients/request
- **Broadcast:** ขึ้นอยู่กับ plan
- **Rate Limit:** 1,000 requests/second per endpoint

**การตรวจสอบ LINE Quota:**
```bash
# ดู Quota ผ่าน API
curl -X GET \
  'https://api.line.me/v2/bot/message/quota' \
  -H 'Authorization: Bearer YOUR_CHANNEL_ACCESS_TOKEN'

# Response:
# {
#   "type": "limited",
#   "value": 500
# }
```

**Dashboard สำหรับ Monitoring:**
- LINE Developers Console - ดู message statistics
- n8n Execution logs - ดู workflow performance
- Google Sheets - ดูจำนวน records ที่ process

---

## 📚 Resources

### เอกสารอ้างอิง
- [n8n Documentation](https://docs.n8n.io/)
- [Google Sheets API](https://developers.google.com/sheets/api)
- [Google Gemini API](https://ai.google.dev/)
- [QuickChart Documentation](https://quickchart.io/documentation/)
- [LINE Messaging API](https://developers.line.biz/en/docs/messaging-api/)
- [LINE Messaging API Reference](https://developers.line.biz/en/reference/messaging-api/)
- [Flex Message Simulator](https://developers.line.biz/flex-simulator/)

### เครื่องมือที่เกี่ยวข้อง
- [Crontab Guru](https://crontab.guru/) - สร้าง cron expressions
- [QuickChart Builder](https://quickchart.io/sandbox/) - ทดสอบกราฟ
- [JSON Formatter](https://jsonformatter.org/) - ตรวจสอบ JSON
- [Flex Message Simulator](https://developers.line.biz/flex-simulator/) - ออกแบบ LINE Flex Message
- [LINE Bot Designer](https://developers.line.biz/console/) - จัดการ LINE Bots

### Community
- [n8n Community Forum](https://community.n8n.io/)
- [n8n Discord](https://discord.gg/n8n)
- [Stack Overflow - n8n tag](https://stackoverflow.com/questions/tagged/n8n)

---

## 🤝 การสนับสนุน

### หากพบปัญหา
1. ตรวจสอบ [การแก้ไขปัญหา](#การแก้ไขปัญหา) ในเอกสารนี้
2. ค้นหาใน [n8n Community Forum](https://community.n8n.io/)
3. สร้าง Issue ใน GitHub repository
4. ติดต่อทีมพัฒนา

### การมีส่วนร่วม
- 🐛 รายงาน bugs
- 💡 แนะนำฟีเจอร์ใหม่
- 📝 ปรับปรุงเอกสาร
- 🔧 ส่ง pull requests

---

## ❓ FAQ (คำถามที่พบบ่อย)

### เกี่ยวกับ LINE Messaging API

**Q: ทำไมข้อความ LINE ไม่ส่งถึง?**
A: ตรวจสอบ:
1. User ต้องเพิ่มบอทเป็นเพื่อนก่อน (สำหรับ 1-on-1)
2. บอทต้องอยู่ใน Group (สำหรับส่งไป group)
3. User ID หรือ Group ID ถูกต้อง
4. Channel Access Token ยังใช้งานได้

**Q: แตกต่างจาก LINE Notify อย่างไร?**
A: LINE Messaging API สามารถส่ง rich content หลายรูปแบบ และทำ interactive bot ได้ แต่มี quota จำกัด (500 push/month สำหรับ free plan) ขณะที่ LINE Notify ส่งได้แค่ text + 1 รูป แต่ไม่จำกัด

**Q: หา User ID ได้จากไหน?**
A: ต้องตั้งค่า Webhook และให้ user ส่งข้อความมาที่บอท จะได้ User ID จาก webhook event หรือใช้เครื่องมือ [LINE Bot Designer](https://developers.line.biz/console/)

**Q: ค่าใช้จ่ายเท่าไหร่?**
A: 
- Free plan: 500 push messages/month
- Paid plan: เริ่มต้น 1,000 บาท/เดือน สำหรับ unlimited push messages
- Reply messages: ฟรีไม่จำกัด (ใช้ได้เมื่อ user ส่งข้อความมาก่อน)

**Q: สามารถส่งไปหลาย User พร้อมกันได้ไหม?**
A: ได้! ใช้ Multicast API:
```javascript
{
  "to": ["USER_ID_1", "USER_ID_2", "USER_ID_3"],
  "messages": [...]
}
```
หรือใช้ Broadcast API สำหรับส่งให้ทุกคนที่เป็นเพื่อน

**Q: Flex Message ทำให้ข้อความสวยงามขนาดไหน?**
A: Flex Message สามารถสร้าง UI ที่สวยงามแบบ card-based พร้อม buttons, images, และ dynamic layout ได้ ดูตัวอย่างที่ [Flex Message Simulator](https://developers.line.biz/flex-simulator/)

**Q: กราฟจาก QuickChart แสดงใน LINE ได้ไหม?**
A: ได้! QuickChart generate URL ที่เป็น HTTPS และเข้าถึงได้สาธารณะ ทำให้ LINE สามารถแสดงรูปได้ทันที ไม่ต้อง host เอง

### เกี่ยวกับ Workflow

**Q: Workflow รันช้า ทำอย่างไร?**
A: 
1. ลดจำนวนข้อมูลที่ process
2. ใช้ pagination สำหรับข้อมูลมาก
3. Cache ผลลัพธ์บางส่วน
4. Optimize JavaScript code

**Q: สามารถส่งรายงานแบบ Real-time ได้ไหม?**
A: ได้! เปลี่ยนจาก Schedule Trigger เป็น Webhook Trigger และเรียก workflow ทุกครั้งที่มีการขายใหม่

**Q: ต้องการเพิ่มผู้รับอีกคน ทำอย่างไร?**
A: แก้ไขใน node "Format LINE Message" เปลี่ยนจากส่งไป 1 user เป็น loop ส่งหลาย users หรือใช้ Multicast API

