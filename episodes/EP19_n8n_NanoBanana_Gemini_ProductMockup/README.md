https://youtu.be/TC5sTjpcEws?si=-72mE3ax9GKUn3Yv

---------------//// node : gemini image to Fabric type ////-------------------
--//url:generative//--

https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-exp-image-generation:generateContent

--// code json //--
{
  "contents": [
    {
      "parts": [
        {
          "text": "ช่วยเขียน prompt สร้างภาพ คุณจะเห็นสามภาพ ภาพแรกเป็นคน ภาพที่สองเป็นเสื้อ ภาพที่สามเป็นรองเท้า ให้วิเคราะห์แล้วสร้างภาพที่คนในภาพสวมเสื้อนี้ และรองเท้า เสื้อควรพอดีกับลำตัวและไหล่ มีรอยพับและเงาที่เป็นธรรมชาติ โลโก้หรือรายละเอียดบนเสื้อต้องชัดเจน ให้สไตล์ภาพสมจริงและสวยงาม แสงและเงาให้ถูกต้อง ใส่พื้นหลังตามสไตล์ของเสื้อ เหมาะสำหรับใช้ในร้านค้าออนไลน์ ตอบเป็นข้อความธรรมดา  ห้ามมี: อักษรพิเศษ อธิบายแบบละเอียดว่าผลลัพธ์เป็นอย่างไร ขึ้นต้นด้วย สร้างภาพ"
        },
        {
          "inline_data": {
            "mime_type": "image/jpeg",
            "data": "{{ $('Upload__shirt').item.json.data }}"
          }
        },
        {
          "inline_data": {
            "mime_type": "image/jpeg",
            "data": "{{ $('Upload__shoe').item.json.data }}"
          }
        },
        {
          "inline_data": {
            "mime_type": "image/jpeg",
            "data": "{{ $json.data }}"
          }
        }
      ]
    }
  ]
}


---------------//// node : code ////-------------------

// รับข้อความจาก input
const prompt = $input.first().json.candidates[0].content.parts[0].text|| "";

// ลบอักขระพิเศษ และ \n ให้เหลือแต่ ตัวอักษร ตัวเลข และช่องว่าง
const cleanPrompt = prompt.replace(/[^a-zA-Z0-9ก-๙\s]/g, " ").replace(/\s+/g, " ").trim();

return [
  {
    json: {
      prompt: cleanPrompt
    }
  }
];

---------------//// node : openrouter ////-------------------
--//url:openrouter//--

https://openrouter.ai/api/v1/chat/completions

--//code json //--
{
  "model": "google/gemini-2.5-flash-image-preview:generateContent",
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "{{ $('Code').item.json.prompt }}"
        },
        {
          "type": "image_url",
          "image_url": {
            "url": "data:image/jpeg;base64,{{ $json['Extract from File'] }}"
          }
        },
        {
          "type": "image_url",
          "image_url": {
            "url": "data:image/jpeg;base64,{{ $json.Upload__shoe }}"
          }
        },
        {
          "type": "image_url",
          "image_url": {
            "url": "data:image/jpeg;base64,{{ $json[' Upload__shirt'] }}"
          }
        }
      ]
    }
  ]
}
