https://youtu.be/Uri9cQHIks0?si=OiIPRXPJ6LHS70_J

#เปิดใช้งาน vector extension

CREATE EXTENSION IF NOT EXISTS vector;


 ---------///////////////////---------------

สร้างตาราง documents สำหรับเก็บข้อความและ Embedding

CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  content TEXT,
  embedding VECTOR(3072),
  metadata JSONB DEFAULT '{}'::jsonb
);

