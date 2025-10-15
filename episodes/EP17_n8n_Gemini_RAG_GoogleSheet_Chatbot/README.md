https://youtu.be/PvsuxwmQywc?si=xm9Kzgx1PgNEjLbA

---------------------////// Node AI Agent classify  /////////--------------------------

--#Prompt (User Message)#--

คุณคือระบบจำแนกข้อความสำหรับร้านกาแฟ

**หน้าที่ของคุณ:** 
ตัดสินใจว่า ข้อความจากผู้ใช้ต้องใช้ RAG หรือเป็นคำถามทั่วไป

**กฎการตัดสินใจ:**
1. ถ้าคำถามต้องใช้ข้อมูลเฉพาะจากฐานข้อมูลหรือเอกสาร (เวลาเปิด-ปิด, เมนู, ราคา, ส่วนผสม, โปรโมชั่น) → ตอบ "RAG"
2. ถ้าเป็นทักทาย แสดงความรู้สึก หรือความคิดเห็นทั่วไป → ตอบ "ทั่วไป"
3. ถ้ามี keyword เช่น "เปิด", "เวลา", "เมนู", "ราคา", "ส่วนผสม", "โปรโมชั่น" → ตอบ "RAG"

**สิ่งสำคัญ:**
- ข้อความ input คือ: `{{ $json.body.events[0].message.text }}`
- ตอบกลับได้แค่หนึ่งคำเท่านั้น: "RAG" หรือ "ทั่วไป"
- ห้ามใส่คำอื่น ๆ เช่น "นี่คือคำตอบ", "คำตอบคือ", หรือคำอธิบายเพิ่มเติม

**ตัวอย่างการตัดสินใจ:**
- "ร้านเปิดกี่โมง?" → RAG
- "เมนูมีอะไรบ้าง?" → RAG
- "กาแฟรสชาติยังไง?" → ทั่วไป
- "สวัสดีครับ" → ทั่วไป

---------------------////// Node AI Agent general  /////////--------------------------

--#Prompt (User Message)#--

คุณคือผู้ช่วยที่สุภาพ เป็นมิตร และเหมาะสำหรับผู้ชาย

ตอบคำถามทั่วไป 
ใช้น้ำเสียงสุภาพ เป็นมิตร แต่ตรงไปตรงมา
ถ้าเป็นไปได้ ให้ตัวอย่างหรือเคล็ดลับ
ถ้าไม่รู้คำตอบ ให้ตรงไปตรงมาและแนะนำวิธีหาข้อมูล

คำถาม: "{{ $('only message').item.json.body.events[0].message.text }}"

กรุณาตอบคำถาม ทันที ไม่ต้องทักทาย

---------------------////// Node AI Agent RAG  /////////--------------------------

--#Prompt (User Message)#--
คุณคือผู้ช่วยตอบแชทอย่างเป็นทางการใน LINE ของร้านกาแฟ
หน้าที่ของคุณคือช่วยเหลือผู้ใช้ด้วยข้อมูลที่ถูกต้องจากร้าน โดย ไม่แต่งเติมเกินจริง

ข้อความจากผู้ใช้:
`{{ $('only message').item.json.body.events[0].message.text }}`

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
   - แบ่งข้อความชัดเจน
   - ปิดท้ายทุกประโยคด้วยคำว่า "ครับ"

4. กฎการตอบ

   - ❌ ห้ามเดาหรือแต่งข้อมูลที่ไม่มีจริง (ยกเว้นแก้พิมพ์ผิด)
   - ❓ ถ้าคำถามไม่เกี่ยวข้องกับร้าน ให้ตอบ: "ขออภัยในความไม่สะดวกครับ"
   - ✅ ถ้าไม่พบข้อมูล ให้ตอบ: "ขอทวนคำถามอีกครั้งได้ไหมครับ \n ร้านของเราคือร้านกาแฟ"
   - ❓ ถ้าข้อมูลบางส่วนไม่ชัดเจน ให้เสนอข้อมูลใกล้เคียงพร้อมถามกลับว่าใช่สิ่งที่ต้องการหรือไม่
   - ❌ ห้ามเอ่ยถึงชื่อฐานข้อมูลหรือกระบวนการค้นหาในการตอบ

---------------------////// Node Clean text  /////////--------------------------

// =======================
// 1. รับ input
// =======================
let rawData = $input.first().json.output ?? $input.first().json;

// =======================
// 2. ฟังก์ชันทำความสะอาด string
// =======================
function cleanString(str) {
  if (!str) return "";
  // ลบ " , * และ /
  let cleanText = str.replace(/["*\/]/g, '');
  // ลบ control chars ยกเว้น newline (\x0A)
  cleanText = cleanText.replace(/[\x00-\x08\x0B\x0C\x0E-\x1F\x7F]/g, ""); 
  // แปลง literal \n เป็น newline จริง
  cleanText = cleanText.replace(/\\n/g, '\n');
  return cleanText.trim();
}

// =======================
// 3. แปลง object เป็นข้อความ readable
// =======================
function objectToText(obj) {
  if (!obj) return "";
  return Object.entries(obj)
    .map(([key, value]) => {
      if (typeof value === "object") {
        return `${key}: ${JSON.stringify(value).replace(/\//g, '')}`;
      }
      return `${key}: ${value}`;
    })
    .join('\n');
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
// 5. แยกข้อความเป็น array ของบรรทัด
// =======================
let lines = displayText.split('\n').filter(line => line.trim() !== "");

// =======================
// 6. สร้าง array ของ text objects สำหรับ Flex
// =======================
let textContents = lines.map(line => ({
  type: "text",
  text: line,
  size: "sm",
  color: "#555555",
  wrap: true
}));

// =======================
// 7. return replyToken + textContents
// =======================
return [{
  json: {
    replyToken: $('only message').first().json.body.events[0].replyToken,
    textContents: JSON.stringify(textContents)
  }
}];

---------------------////// Node send general to LINE  /////////--------------------------
-----------# JSON #--------------

{
  "replyToken": "{{ $json.replyToken }}",
  "messages": [
    {
      "type": "flex",
      "altText": "ข้อมูลทั่วไป",
      "contents": {
        "type": "bubble",
        "body": {
          "type": "box",
          "layout": "vertical",
          "spacing": "md",
          "contents": [
            {
              "type": "text",
              "text": "📌 ข้อมูลทั่วไป",
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
              "contents": {{ $json.textContents }}
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

---------------------////// Node code to google sheet  /////////--------------------------

// ----- 1️⃣ ดึงข้อมูล LINE Webhook -----
const webhookData = $('Webhook').first().json;
const events = webhookData.body?.events || webhookData.events || [];

// ----- 2️⃣ Mapping alias → ชื่อจริง -----
const menuAliases = {
  "Hot Drinks": {
    "ลาเต้": "ลาเต้",
    "คาปูชิโน": "คาปูชิโน",
    "มอคค่า": "มอคค่า",
    "กาแฟร้อน": "กาแฟร้อน",
    "อเมริกาโน่": "อเมริกาโน่",
    "เอสเพรสโซ": "เอสเพรสโซ",
    "ช็อคโกแลต": "ช็อคโกแลต",
    "มัทฉะร้อน": "มัทฉะร้อน"
  },
  "Cold Drinks": {
    "ลาเต้เย็น": "ลาเต้เย็น",
    "คาปูชิโนเย็น": "คาปูชิโนเย็น",
    "มอคค่าเย็น": "มอคค่าเย็น",
    "อเมริกาโนเย็น": "อเมริกาโนเย็น",
    "ชาเขียวเย็น": "ชาเขียวเย็น",
    "ชาไทยเย็น": "ชาไทยเย็น",
    "ชามะนาวเย็น": "ชามะนาวเย็น",
    "น้ำผึ้งมะนาว": "น้ำผึ้งมะนาว",
    "โซดามะนาว": "โซดามะนาว"
  },
  "Dessert": {
    "เค้ก": "เค้ก",
    "ครัวซองต์": "ครัวซองต์",
    "บราวนี่": "บราวนี่",
    "ชีสเค้ก": "ชีสเค้ก",
    "มัฟฟินบลูเบอร์รี่": "มัฟฟินบลูเบอร์รี่",
    "คุกกี้ช็อกชิพ": "คุกกี้ช็อกชิพ"
  }
};

// ----- 3️⃣ Levenshtein distance -----
function levenshtein(a, b) {
  const matrix = Array.from({ length: b.length + 1 }, (_, i) => [i]);
  for (let j = 0; j <= a.length; j++) matrix[0][j] = j;

  for (let i = 1; i <= b.length; i++) {
    for (let j = 1; j <= a.length; j++) {
      if (b[i - 1] === a[j - 1]) matrix[i][j] = matrix[i - 1][j - 1];
      else
        matrix[i][j] = Math.min(
          matrix[i - 1][j] + 1,
          matrix[i][j - 1] + 1,
          matrix[i - 1][j - 1] + 1
        );
    }
  }
  return matrix[b.length][a.length];
}

// ----- 4️⃣ ฟังก์ชัน match alias เต็ม -----
function matchMenuStrict(text) {
  const cleanInput = text.trim().replace(/\s+/g, " ").replace(/\n|\t/g, "");
  for (const [category, aliases] of Object.entries(menuAliases)) {
    if (Object.prototype.hasOwnProperty.call(aliases, cleanInput)) {
      return { dishName: aliases[cleanInput], category };
    }
  }
  return null;
}

// ----- 5️⃣ ฟังก์ชัน fuzzy suggestion -----
function getSuggestedDish(text) {
  const cleanInput = text.trim().replace(/\s+/g, " ").replace(/\n|\t/g, "");
  let bestMatch = null;
  let bestCategory = null;
  let bestScore = Infinity;

  for (const [category, aliases] of Object.entries(menuAliases)) {
    for (const [alias, name] of Object.entries(aliases)) {
      // เช็ค prefix ก่อน
      if (alias.startsWith(cleanInput)) {
        return { suggestedDishName: name, suggestedCategory: category };
      }

      // Levenshtein normalized
      const distance = levenshtein(cleanInput, alias) / Math.max(cleanInput.length, alias.length);
      if (distance < bestScore) {
        bestScore = distance;
        bestMatch = name;
        bestCategory = category;
      }
    }
  }

  return bestMatch ? { suggestedDishName: bestMatch, suggestedCategory: bestCategory } : null;
}


// ----- 6️⃣ classifyMessage แบบ hybrid -----
function classifyMessage(text) {
  const t = text.toLowerCase();
  let result = {
    category: "General",
    intent: "General",
    sentiment: "Neutral",
    dishName: "",
    suggestedDishName: "",
    temperature: "",
    size: "",
    options: "",
    quantity: 0,
    faqTopic: "",
    faqAnswerSource: "",
    feedbackType: "",
    feedbackComment: "",
    ragSource: "",
    isParsed: false,
    error: ""
  };

  try {
    // Intent
    if (/เอา|ขอ|สั่ง|อยากได้/.test(t)) result.intent = "Order";

    // FAQ
    if (/ราคา|กี่โมง|เมนู|ที่อยู่|เบอร์/.test(t)) {
      result.category = "FAQ";
      result.intent = "Inquiry";
      result.faqTopic = "GeneralInfo";
      result.isParsed = true;
    }

    // Strict match
    const match = matchMenuStrict(text);
    if (match) {
      result.category = match.category;
      result.intent = "OrderIntent";
      result.dishName = match.dishName;
      result.suggestedDishName = match.dishName;
      result.size = "Regular";
      result.quantity = 1;
      if (match.category === "Hot Drinks") result.temperature = "ร้อน";
      else if (match.category === "Cold Drinks") result.temperature = "เย็น";
      result.isParsed = true;
    } else {
      // Fuzzy match
      const suggestion = getSuggestedDish(text);
      if (suggestion) {
        result.suggestedDishName = suggestion.suggestedDishName;
        result.dishName = suggestion.suggestedDishName;
        result.category = suggestion.suggestedCategory || "General";
        result.size = "Regular";
        result.quantity = 1;
        if (suggestion.suggestedCategory === "Hot Drinks") result.temperature = "ร้อน";
        else if (suggestion.suggestedCategory === "Cold Drinks") result.temperature = "เย็น";
        result.isParsed = true;
      }
    }

    // Feedback
    if (/ไม่อร่อย|ขมไป|หวานไป|ช้า/.test(t)) {
      result.category = "Feedback";
      result.intent = "Feedback";
      result.feedbackComment = text;
      result.sentiment = "Negative";
      result.isParsed = true;
    }

    // Options
    if (/ไม่ใส่น้ำตาล/.test(t)) {
      result.options = "ไม่ใส่น้ำตาล";
    }

  } catch (err) {
    result.error = err.message;
  }

  return result;
}

// ----- 7️⃣ ประมวลผลทุก event -----
const results = [];

for (const event of events) {
  // ตรวจว่ามี text message
  const messageText = event.message?.text || "";
  if (!messageText) continue;

  const userId = event.source?.userId || event.source?.groupId || "U0000";
  const messageId = event.message?.id || "";
  const replyToken = event.replyToken || "";
  const timestamp = event.timestamp ? new Date(event.timestamp).toISOString() : new Date().toISOString();
  const sessionId = `${userId}_${Date.now()}`; // ใช้ timestamp เต็มเพื่อไม่ซ้ำ

  const classified = classifyMessage(messageText);

  results.push({
    json: {
      timestamp,
      sessionId,
      userId,
      messageId,
      replyToken,
      originalText: messageText,
      category: classified.category,
      intent: classified.intent,
      sentiment: classified.sentiment,
      dishName: classified.dishName,
      suggestedDishName: classified.suggestedDishName,
      temperature: classified.temperature,
      size: classified.size,
      options: classified.options,
      quantity: classified.quantity,
      faqTopic: classified.faqTopic,
      faqAnswerSource: classified.faqAnswerSource,
      feedbackType: classified.feedbackType,
      feedbackComment: classified.feedbackComment,
      ragSource: classified.ragSource,
      isParsed: classified.isParsed,
      error: classified.error
    }
  });
}

return results;



