
https://youtu.be/sJ_icQ6R2Hs?si=pbTwjrthoSo2ENtW

#node Process Context

<document> 
{{ $('AI Agent1').item.json.output }}
</document> 

<chunk> 
{{ $('Split into chunks').item.json.texts }}
</chunk> 

โปรดดำเนินการดังต่อไปนี้:
- วิเคราะห์ chunk ด้านล่างนี้ว่ามาจากหมวดหมู่ใดในเอกสาร (เช่น กาแฟร้อน, กาแฟเย็น, ชาและเครื่องดื่ม, เบเกอรี่)
- เขียนสรุปบริบท (context) สั้น ๆ โดยระบุชื่อหมวดหมู่และประเภทข้อมูล (เช่น ตารางราคา, รายการเมนู)
- คืนค่า chunk ดั้งเดิมทุกประการ ยกเว้นกรณี:
    * มีชื่อเมนู, ราคา, หรือคำอธิบายที่หายไป ให้เติมจากเอกสารฉบับเต็ม
    * chunk ถูกตัดกลางตาราง ให้เติมแถวของตารางให้ครบ
- หากมีข้อมูลซ้ำ ให้คงไว้เพียงครั้งเดียวใน chunk
- ห้ามเปลี่ยนเนื้อหาที่ถูกต้องอยู่แล้ว

**รูปแบบการตอบกลับ:**
[บริบทโดยย่อ] : [chunk ดั้งเดิม หรือฉบับแก้ไขถ้าจำเป็น]

**คำตอบต้องอยู่ในภาษาไทยเท่านั้น** และ **อย่าเพิ่มคำอธิบายอื่นนอกเหนือจากที่ระบุ**

---------------//////// END Workflow Rag Gemini Ollama Supabase Vector ///////////----------------------




---------------//////// Workflow RAG + LINE ///////////----------------------


------//// Node AI Agent ////-------------------

คุณคือผู้ช่วยตอบแชทอย่างเป็นทางการใน LINE ของร้านกาแฟ
หน้าที่ของคุณคือช่วยเหลือผู้ใช้ด้วยข้อมูลที่ถูกต้องจากร้าน โดย ไม่แต่งเติมเกินจริง

ข้อความจากผู้ใช้: `{{ $json.body.events[0].message.text }}`

ขั้นตอนการทำงาน:

1. วิเคราะห์และทำความเข้าใจคำถาม
   - ตรวจสอบการพิมพ์ผิดหรือข้อความที่ตกหล่น แล้วแก้ไขให้ถูกต้องครับ
   - ตีความความหมายที่เกี่ยวข้องกับร้าน เช่น สินค้า เมนู โปรโมชั่น บริการ เวลาทำการ หรือคำถามอื่นๆ

2. ตรวจสอบข้อมูลใน Supabase Vector Store ก่อนตอบทุกครั้ง
   - ใช้ข้อความที่ปรับแล้วค้นหาในฐานข้อมูลของร้าน
   - ถ้าพบข้อมูลที่ตรงกัน ให้ใช้ข้อมูลนั้นในการตอบ
   - ถ้าไม่ตรง ให้ค้นหาสิ่งที่ใกล้เคียงที่สุดแล้วตรวจสอบอีกครั้ง

3. สร้างคำตอบ
   - เรียบเรียงใหม่เป็นภาษาธรรมชาติ อ่านลื่นไหล ฟังง่าย
   - จัดลำดับประโยคเพื่อความสละสลวยโดยไม่เปลี่ยนความหมาย
   - หลีกเลี่ยงคำซ้ำและโครงสร้างซ้ำซาก
   - ปิดท้ายทุกประโยคด้วยคำว่า "ครับ"

4. กฎการตอบ

   - ❌ ห้ามเดาหรือแต่งข้อมูลที่ไม่มีจริง (ยกเว้นแก้พิมพ์ผิด)
   - ❓ ถ้าคำถามไม่เกี่ยวข้องกับร้าน ให้ตอบ: "ขออภัยในความไม่สะดวกครับ"
   - ✅ ถ้าไม่พบข้อมูล ให้ตอบ: "ขอทวนคำถามอีกครั้งได้ไหมครับ \n ร้านของเราคือร้านกาแฟ"
   - ❓ ถ้าข้อมูลบางส่วนไม่ชัดเจน ให้เสนอข้อมูลใกล้เคียงพร้อมถามกลับว่าใช่สิ่งที่ต้องการหรือไม่
   - ❌ ห้ามเอ่ยถึงชื่อฐานข้อมูลหรือกระบวนการค้นหาในการตอบ

------////  end Node AI Agent ////-------------------




------//// Supabase Vector Store ////-------------------
#Description
ใช้ค้นหาข้อมูลจากฐานข้อมูลเวกเตอร์ที่อยู่ใน Supabase เท่านั้น เครื่องมือนี้จะรับคำค้นหาแล้วคืนข้อความที่ใกล้เคียงที่สุดจากเอกสารในระบบ ห้ามสร้างคำตอบจากความรู้ของคุณเองโดยเด็ดขาด หากไม่พบข้อมูลที่ตรงกับคำถาม ให้บอกผู้ใช้ว่า "นั่นไม่ใช่คำที่มีอยู่ในฐานข้อมูล"

------//// end Supabase Vector Store ////-------------------






---------//// Node code ////-------------

// =======================
// 1. รับ input
// =======================
let rawData = $input.first().json.output ?? $input.first().json;

// =======================
// 2. ฟังก์ชันทำความสะอาด string (ไม่ลบ \n)
// =======================
function cleanString(str) {
  if (!str) return "";
  // ลบ " และ * แต่เก็บ \n ไว้
  let cleanText = str.replace(/["*]/g, '');
  // ลบ control chars ยกเว้น \n (\x0A)
  cleanText = cleanText.replace(/[\x00-\x08\x0B\x0C\x0E-\x1F\x7F]/g, ""); 
  return cleanText;
}

// =======================
// 3. ฟังก์ชันแปลง object เป็นข้อความ readable
// =======================
function objectToText(obj) {
  if (!obj) return "";
  return Object.entries(obj)
    .map(([key, value]) => {
      if (typeof value === "object") {
        return `${key}: ${JSON.stringify(value)}`;
      }
      return `${key}: ${value}`;
    })
    .join('\n'); // ใช้ \n แทน space → จะได้หลายบรรทัด
}

// =======================
// 4. กำหนด displayText
// =======================
let displayText = "";
if (typeof rawData === "string") {
  displayText = cleanString(rawData);
} else if (typeof rawData === "object") {
  displayText = objectToText(rawData);
} else {
  displayText = String(rawData);
}

// =======================
// 5. แยกข้อความตาม \n
// =======================
let lines = displayText.split('\n').filter(line => line.trim() !== "");

// =======================
// 6. สร้าง array ของ text objects สำหรับ Flex Message
// =======================
let textContents = lines.map(line => ({
  type: "text",
  text: line,
  size: "sm",
  color: "#555555",
  wrap: true
}));

// =======================
// 7. ส่งออก Flex Message
// =======================
// สมมติ displayText แยก \n แล้วเป็น textContents
return [{
  json: {
    replyToken: $('The fields we want').first().json.body.events[0].replyToken,
    messages: [
      {
        type: "flex",
        altText: "ข้อมูลจากระบบ",
        contents: {
          type: "bubble",
          body: {
            type: "box",
            layout: "vertical",
            spacing: "md",
            contents: [
              {
                type: "text",
                text: "📌 ข้อมูลระบบ",
                weight: "bold",
                size: "xl",
                color: "#1DB446"
              },
              {
                type: "separator",
                margin: "sm",
                color: "#CCCCCC"
              },
              {
                type: "box",
                layout: "vertical",
                margin: "md",
                spacing: "sm",
                contents: textContents
              },
              {
                type: "separator",
                margin: "md",
                color: "#EEEEEE"
              },
              {
                type: "box",
                layout: "vertical",
                margin: "md",
                contents: [
                  {
                    type: "text",
                    text: "ℹ️ หากต้องการรายละเอียดเพิ่มเติม กรุณาติดต่อฝ่ายสนับสนุน",
                    size: "xs",
                    wrap: true,
                    color: "#AAAAAA"
                  }
                ]
              }
            ]
          }
        }
      }
    ],
    // ✅ เพิ่ม textContents ให้ส่งกลับด้วย
    textContents: JSON.stringify(textContents, null, 2)
  }
}];


----------------/// node Reply to line ///--------------------

URL: https://api.line.me/v2/bot/message/reply

JSON : 
---------------------------------------------
{
  "replyToken": "{{$json["replyToken"]}}",
  "messages": [
    {
      "type": "flex",
      "altText": "ข้อมูลจากระบบ",
      "contents": {
        "type": "bubble",
        "body": {
          "type": "box",
          "layout": "vertical",
          "spacing": "md",
          "contents": [
            {
              "type": "text",
              "text": "📌 ข้อมูลระบบ",
              "weight": "bold",
              "size": "xl",
              "color": "#1DB446"
            },
            {
              "type": "separator",
              "margin": "sm",
              "color": "#CCCCCC"
            },
            {
              "type": "box",
              "layout": "vertical",
              "margin": "md",
              "spacing": "sm",
              "contents": {{ $json["textContents"]}}
            },
            {
              "type": "separator",
              "margin": "md",
              "color": "#EEEEEE"
            },
            {
              "type": "box",
              "layout": "vertical",
              "margin": "md",
              "contents": [
                {
                  "type": "text",
                  "text": "ℹ️ หากต้องการรายละเอียดเพิ่มเติม กรุณาติดต่อฝ่ายสนับสนุน",
                  "size": "xs",
                  "wrap": true,
                  "color": "#AAAAAA"
                }
              ]
            }
          ]
        }
      }
    }
  ]
}
