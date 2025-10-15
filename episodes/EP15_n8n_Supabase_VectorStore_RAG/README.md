# เชื่อมต่อ Supabase Vector Store กับ n8n ทำ RAG Workflow ง่าย ๆ
---

🎥 วิดีโอสอน: 
[![Watch the video](https://github.com/novelbiz/AI_Automation/blob/main/assets/thumbnail/Supabase%20Vector%20Store.webp)](https://youtu.be/Uri9cQHIks0?si=mbey57k2AEeabrHW)

---
## สรุปสั้น ๆ
เอกสารนี้อธิบายการเปิดใช้งาน `pgvector` (Postgres extension ชื่อ `vector`) บนฐานข้อมูล PostgreSQL / Supabase แล้วสร้างตาราง `documents` ที่เก็บ `content` (ข้อความ), `embedding` (vector) และ `metadata` (JSONB) เพื่อใช้เป็น Vector Store สำหรับงาน RAG / semantic search / similarity search ฯลฯ

---

## ข้อกำหนดเบื้องต้น (Prerequisites)
1. PostgreSQL หรือ Supabase ที่รองรับการเปิดใช้ extension
2. ทราบขนาดมิติ (dimension) ของ embedding ที่จะเก็บ เช่น `3072`

---

## 1) เปิดใช้งาน `vector` extension
```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

หากใช้ Supabase สามารถเปิดได้ที่หน้า **Database → Extensions → vector → Enable**

---

## 2) สร้างตาราง `documents`
```sql
CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  content TEXT,
  embedding VECTOR(3072),
  metadata JSONB DEFAULT '{}'::jsonb
);
```

### คำอธิบาย
- **id**: รหัสเอกสาร (UUID)
- **content**: ข้อความต้นฉบับ
- **embedding**: ข้อมูลเวกเตอร์ขนาด 3072 มิติ
- **metadata**: ข้อมูลเพิ่มเติม เช่น source หรือ URL

---

## 3) เพิ่ม Index สำหรับค้นหาเร็วขึ้น (HNSW)
```sql
CREATE INDEX ON documents
USING hnsw (embedding vector_cosine_ops);
```

---

## 4) ตัวอย่างการ Insert ข้อมูล
```sql
INSERT INTO documents (content, embedding, metadata)
VALUES (
  'นี่คือข้อความตัวอย่าง',
  '[0.0123, 0.4567, -0.0034]'::vector,
  '{"source": "web", "url": "https://example.com"}'::jsonb
);
```

---

## 5) ตัวอย่างการค้นหา (Query)
```sql
SELECT id, content, metadata,
       embedding <=> '[0.1, 0.2, ...]'::vector AS cosine_distance
FROM documents
ORDER BY embedding <=> '[0.1, 0.2, ...]'::vector
LIMIT 5;
```

---

## 6) ปัญหาที่พบบ่อย
- หากขึ้น `permission denied to create extension "vector"` ต้องให้ผู้ดูแลระบบเปิดให้ก่อน
- หาก `gen_random_uuid()` ใช้ไม่ได้ ให้รัน:
  ```sql
  CREATE EXTENSION IF NOT EXISTS "pgcrypto";
  ```

---

## 7) การใช้งานร่วมกับ n8n

### Workflow ตัวอย่าง
1. **Webhook Node** → รับข้อความจากผู้ใช้ (LINE, Web, Telegram ฯลฯ)
2. **Function Node** → เตรียมข้อความสำหรับ Embedding
3. **AI Node** → สร้าง Embedding (Google Gemini หรือ Ollama)
4. **HTTP Request Node** → เรียก Supabase RPC `match_documents` เพื่อค้นหาข้อความใกล้เคียง
5. **AI Node** → สร้างคำตอบจากข้อมูลที่ค้นพบ
6. **HTTP Request Node** → ส่งคำตอบกลับผู้ใช้ (เช่น ผ่าน LINE API)

### ตัวอย่าง SQL function: `match_documents`
```sql
create or replace function match_documents(
  query_embedding vector(3072),
  match_threshold float,
  match_count int
)
returns table(
  id uuid,
  content text,
  metadata jsonb,
  similarity float
)
language sql stable as $$
  select
    id,
    content,
    metadata,
    1 - (embedding <=> query_embedding) as similarity
  from documents
  where 1 - (embedding <=> query_embedding) > match_threshold
  order by embedding <=> query_embedding
  limit match_count;
$$;
```
---
# เคสปัญหาพบบ่อย & แนวทางแก้ไข (pgvector / PostgreSQL)

## 1. ไม่สามารถสร้าง extension — `permission denied to create extension "vector"`
- **สาเหตุ:** ผู้ใช้ที่เชื่อมต่อไม่ใช่ superuser หรือผู้ให้บริการ DB บล็อกการสร้าง extension
- **แนวทางแก้ไข:** ให้ผู้ดูแลสร้าง extension หรือใช้ image/docker ที่มี pgvector ติดตั้งไว้แล้ว
- **อ้างอิง:** Reddit +1

## 2. เรียกใช้ `gen_random_uuid()` แล้ว error
- **แนวทางแก้ไข:** ต้องเปิด `pgcrypto` extension หรือใช้ `uuid-ossp` แล้วใช้ `uuid_generate_v4()`
```sql
CREATE EXTENSION IF NOT EXISTS "pgcrypto";
-- or
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

## 3. ขนาดมิติไม่ตรงกับ embeddings ที่สร้าง
- **สาเหตุ:** สร้างคอลัมน์ `vector(3072)` แต่ embeddings ของโมเดลเป็น `1536`
- **แนวทางแก้ไข:** ให้กำหนดขนาดให้ตรงกันหรือเก็บ vector แบบไดนามิก (ต้องมีการจัดการ) — ปกติกำหนดค่าให้ตรงกับโมเดลเสมอ
- **อ้างอิง:** GitHub

## 4. `extension "vector" already exists`
- **สาเหตุ:** Extension ถูกเปิดแล้ว
- **แนวทางแก้ไข:** สามารถข้ามขั้นตอนการรัน `CREATE EXTENSION` ได้เลย
---
### แหล่งอ้างอิง
- [pgvector official documentation](https://github.com/pgvector/pgvector)
- [Supabase Vector Store docs](https://supabase.com/docs/guides/ai/pgvector)
- [n8n AI Integration Guide](https://docs.n8n.io/integrations/builtin/ai/)

