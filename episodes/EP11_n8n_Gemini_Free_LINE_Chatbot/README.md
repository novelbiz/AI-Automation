
https://youtu.be/0N3X9lG6n2c?si=p7znzi1guv2PKRmz

#NODE AI Agent
-----------------------
#Prompt (User Message)
นี่คือข้อความที่ผู้ใช้ส่งมาใน LINE: "{{ $json.body.events[0].message.text }}"
โปรดตอบกลับด้วยภาษาที่สุภาพ เป็นกันเอง และอ่านง่าย
ใช้ภาษาธรรมชาติ และสามารถใส่อีโมจิได้ถ้าเหมาะสม
ตอบให้ตรงประเด็น เข้าใจง่าย และเป็นมิตร
-----------------------
#System Message
คุณเป็นผู้ช่วยอัจฉริยะสำหรับการสนทนาใน LINE  
สามารถตอบคำถามได้ทุกเรื่อง ทั้งข้อมูลทั่วไป ความรู้ บทสนทนา หรือช่วยแก้ปัญหา  
ให้ตอบด้วยภาษาที่สุภาพ เป็นกันเอง และเข้าใจง่าย  
สามารถใส่อีโมจิได้ถ้าเหมาะสม  
ห้ามใส่อักขระพิเศษ เช่น \n, \r, \, หรือเครื่องหมายที่ไม่จำเป็นในข้อความตอบกลับ  
ถ้าข้อความจากผู้ใช้ไม่ชัดเจน ให้ถามย้อนเพื่อขอข้อมูลเพิ่ม  
หลีกเลี่ยงการใช้ภาษาที่ไม่เหมาะสมหรือสร้างความขัดแย้ง

--------------------------------------------//////////////-------------------------------------------------------

#NODE Simple Memory
--------------------
#Key
return [{ json: { sessionKey: $crypto.randomBytes(16).toString('hex') } }];

-------------------------------------------///////////////-------------------------------------------------------

#NODE Code
------------------
#JavaScript
// รับข้อมูลจาก Node ก่อนหน้า (string หรือ object)$input.first().json.output
let rawData = $input.first().json.output ?? $input.first().json;

// แปลงเป็น string ถ้ายังไม่ใช่
if (typeof rawData !== 'string') {
  rawData = JSON.stringify(rawData);
}

// ลบ \n, \r, และเครื่องหมาย / ที่อยู่นอก JSON
rawData = rawData.replace(/\r?\n|\r/g, '').replace(/\//g, '');

// ดึงเฉพาะส่วนที่เป็น JSON {...}
const match = rawData.match(/\{[\s\S]*\}/);

if (match) {
  try {
    const cleanJson = JSON.parse(match[0]);
    return [{ json: cleanJson }];
  } catch (err) {
    return [{ json: { error: 'Invalid JSON after cleaning', raw: match[0] } }];
  }
} else {
  return [{ json: { error: 'No JSON found in input', raw: rawData } }];
}

--------------------------------------------/////////////////------------------------------------------------------------

#NODE HTTP Request
-----------------------
#URL reply
https://api.line.me/v2/bot/message/reply
-----------------------
#Body Content
#JSON

{
  "replyToken": "{{ $('Edit Fields').item.json.body.events[0].replyToken }}",
  "messages": [
    {
      "type": "text",
      "text": "{{ $json.raw }}"
    }
  ]
}

