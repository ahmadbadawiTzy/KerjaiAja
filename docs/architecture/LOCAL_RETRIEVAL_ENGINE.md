# Local Retrieval Engine — SQLite FTS5 & BM25
## Mesin Pencarian Leksikal & Ranking Relevansi On-Device

- **Status:** APPROVED & LOCKED
- **Lokasi Dokumen:** `docs/architecture/LOCAL_RETRIEVAL_ENGINE.md`
- **Terkait:** [`KNOWLEDGE_INTELLIGENCE_LAYER.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/KNOWLEDGE_INTELLIGENCE_LAYER.md), [`ADR-015`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-015-LOCAL-RAG-SQLITE-FTS5.md)

---

## 1. Arsitektur Engine & Pilihan Teknologi

Mesin pencarian lokal KerjaiAja dibangun menggunakan modul **SQLite FTS5 (Full-Text Search 5)** yang merupakan ekstensi native pada SQLite di Android OS (API 26+). Pilihan ini memberikan rasio performa-ke-sumber-daya tertinggi:
- **Latensi Kueri:** Rata-rata 4 ms – 18 ms pada database berisi 5.000 entitas.
- **Overhead Memori:** 0 MB model weight, <8 MB RAM saat eksekusi kueri.
- **Dependensi Tambahan:** 0 library eksternal (terintegrasi langsung dengan Room Database Android).
- **Kemampuan Ranking:** Menggunakan fungsi pembobotan matematis **BM25 (*Best Matching 25*)** bawaan SQLite.

---

## 2. Skema Tabel Virtual FTS5

KerjaiAja memisahkan penyimpanan pencarian menjadi dua tabel virtual FTS5 independen untuk menjamin pemisahan domain secara fisik:

### 2.1 Domain A: `tasks_fts` (Productivity Knowledge)
Tabel ini memetakan data tugas mandiri mahasiswa dari tabel utama `tasks`:

```sql
CREATE VIRTUAL TABLE IF NOT EXISTS tasks_fts USING fts5(
    title,
    description,
    subject_name,
    tags,
    content='tasks',
    content_rowid='id',
    tokenize='unicode61 remove_diacritics 1'
);

-- Triggers untuk sinkronisasi otomatis
CREATE TRIGGER IF NOT EXISTS tasks_ai AFTER INSERT ON tasks BEGIN
    INSERT INTO tasks_fts(rowid, title, description, subject_name, tags)
    VALUES (new.id, new.title, new.description, new.subject_name, new.tags);
END;

CREATE TRIGGER IF NOT EXISTS tasks_ad AFTER DELETE ON tasks BEGIN
    INSERT INTO tasks_fts(tasks_fts, rowid, title, description, subject_name, tags)
    VALUES('delete', old.id, old.title, old.description, old.subject_name, old.tags);
END;

CREATE TRIGGER IF NOT EXISTS tasks_au AFTER UPDATE ON tasks BEGIN
    INSERT INTO tasks_fts(tasks_fts, rowid, title, description, subject_name, tags)
    VALUES('delete', old.id, old.title, old.description, old.subject_name, old.tags);
    INSERT INTO tasks_fts(rowid, title, description, subject_name, tags)
    VALUES (new.id, new.title, new.description, new.subject_name, new.tags);
END;
```

### 2.2 Domain B: `academic_fts` (Academic Knowledge)
Tabel ini memetakan potongan materi/dokumen referensi akademik dari tabel `academic_chunks`:

```sql
CREATE VIRTUAL TABLE IF NOT EXISTS academic_fts USING fts5(
    section_title,
    content,
    subject,
    category,
    keywords,
    content='academic_chunks',
    content_rowid='id',
    tokenize='unicode61 remove_diacritics 1'
);

-- Triggers untuk pembaruan indeks materi
CREATE TRIGGER IF NOT EXISTS academic_chunks_ai AFTER INSERT ON academic_chunks BEGIN
    INSERT INTO academic_fts(rowid, section_title, content, subject, category, keywords)
    VALUES (new.id, new.section_title, new.content, new.subject, new.category, new.keywords);
END;

CREATE TRIGGER IF NOT EXISTS academic_chunks_ad AFTER DELETE ON academic_chunks BEGIN
    INSERT INTO academic_fts(academic_fts, rowid, section_title, content, subject, category, keywords)
    VALUES('delete', old.id, old.section_title, old.content, old.subject, old.category, old.keywords);
END;

CREATE TRIGGER IF NOT EXISTS academic_chunks_au AFTER UPDATE ON academic_chunks BEGIN
    INSERT INTO academic_fts(academic_fts, rowid, section_title, content, subject, category, keywords)
    VALUES('delete', old.id, old.section_title, old.content, old.subject, old.category, old.keywords);
    INSERT INTO academic_fts(rowid, section_title, content, subject, category, keywords)
    VALUES (new.id, new.section_title, new.content, new.subject, new.category, new.keywords);
END;
```

---

## 3. Algoritma Ranking BM25 & Pembobotan Kolom

SQLite FTS5 menyediakan fungsi pembobotan relevansi leksikal menggunakan varian fungsi BM25:

$$\text{BM25 Score} = \sum_{i=1}^{N} \text{IDF}(q_i) \cdot \frac{f(q_i, D) \cdot (k_1 + 1)}{f(q_i, D) + k_1 \cdot \left(1 - b + b \cdot \frac{|D|}{\text{avgdl}}\right)}$$

Di mana SQLite FTS5 mengembalikan nilai skor (makin negatif artinya makin relevan jika menggunakan sintaks ranking bawaan, atau diurutkan berdasarkan `bm25(...) ASC`).

### 3.1 Pembobotan Kolom Spesifik (Column Weights)
Tidak semua kolom memiliki bobot yang sama. Judul bab dan kata kunci memiliki signifikansi lebih tinggi daripada isi paragraf:
- **`academic_fts` weights:**
  - `section_title`: Bobot **5.0** (Sangat menentukan konteks spesifik).
  - `keywords`: Bobot **4.0** (Konsep kunci / rumus).
  - `subject`: Bobot **3.0** (Kesesuaian rumpun ilmu).
  - `content`: Bobot **1.0** (Teks penjelasan isi).

Kueri SQL dengan pembobotan kolom di Kotlin Room:
```sql
SELECT 
    c.id, 
    c.document_id,
    c.section_title, 
    c.content, 
    c.page_number,
    c.subject,
    bm25(academic_fts, 5.0, 1.0, 3.0, 2.0, 4.0) AS relevance_rank
FROM academic_chunks c
JOIN academic_fts ON c.id = academic_fts.rowid
WHERE academic_fts MATCH :sanitizedFtsQuery
  AND c.subject = :targetSubject
  AND c.is_published = 1
ORDER BY relevance_rank ASC
LIMIT :topK;
```

---

## 4. Normalisasi Kueri & Query Builder

Sebelum kueri dikirim ke engine SQLite FTS5, teks input mentah (dari pertanyaan teks atau hasil OCR) diproses melalui pipeline normalisasi:

1. **Case-Folding & Diacritics Removal:** Mengubah seluruh huruf menjadi huruf kecil dan membersihkan aksen diakritik.
2. **Pembersihan Karakter Khusus FTS5:** Menghapus karakter operator sintaks FTS5 yang berbahaya (`"`, `*`, `^`, `:`, `NOT`, `AND`, `OR`, `(`, `)`) guna mencegah kueri error (*syntax injection*).
3. **Penyaringan Stopwords Akademik:** Menghapus partikel umum bahasa Indonesia yang tidak bernilai informasi (`dan`, `atau`, `yang`, `untuk`, `pada`, `ke`, `di`, `dari`, `adalah`, `sebagai`, `ini`, `itu`).
4. **Token Min-Length:** Mengabaikan token dengan panjang $<3$ karakter kecuali simbol matematika/fisika yang telah didaftarkan (misal: `pH`, `kg`, `F`, `a`).
5. **Konstruksi Klausa Disjungtif Berbobot:** Menyusun kueri FTS5 dalam bentuk disjungtif `token1 OR token2 OR token3` dengan pembatasan maksimal 8 token paling bermakna.

---

## 5. Evaluasi Semantic Retrieval & Hybrid Search

Arsitektur KerjaiAja merencanakan pengujian bertahap untuk mengevaluasi apakah pencarian semantik padat (*dense vector search*) memberikan peningkatan yang sebanding dengan biaya komputasi yang ditimbulkannya:

```
+-------------------------------------------------------------------------+
|                  EVALUATION PIPELINE: HYBRID SEARCH                     |
|                                                                         |
|  [Query Mahasiswa: "bagaimana hukum gerak newton pada bidang miring?"] |
|                     |                                                   |
|         +-----------+-----------+                                       |
|         |                       |                                       |
|         v                       v                                       |
|  [Track 1: BM25 Lexical]   [Track 2: On-Device Embedding (Opsional)]    |
|  - SQLite FTS5 Match       - Model: EmbeddingGemma / MiniLM (384-d)    |
|  - Latensi: 8 ms           - Latensi: 120 ms (CPU/NPU)                  |
|  - RAM: <5 MB              - RAM: +180 MB                               |
|         |                       |                                       |
|         v                       v                                       |
|  [Top 10 BM25 Chunks]     [Top 10 Vector Chunks]                        |
|         |                       |                                       |
|         +-----------+-----------+                                       |
|                     |                                                   |
|                     v                                                   |
|  [Reciprocal Rank Fusion (RRF) Merger]                                 |
|  RRF_Score = 1/(60 + rank_bm25) + 1/(60 + rank_vector)                  |
|                     |                                                   |
|                     v                                                   |
|  [Final Top 5 Ranked Candidates]                                        |
+-------------------------------------------------------------------------+
```

### Kriteria Pembuktian Manfaat (Proof-of-Value Gate):
Pencarian semantik / Hybrid Search **HANYA AKAN DIADOPSI KE PRODUKSI JIKA**:
1. Mengalami peningkatan metrik **Recall@5** minimal $\ge 15\%$ pada pengujian kueri sinonim/parafrase dibandingkan baseline FTS5 BM25 murni.
2. Konsumsi RAM tambahan pada ponsel 4 GB tidak pernah melebihi 100 MB.
3. Latensi total kueri tetap di bawah ambang batas $200\text{ ms}$.
4. Jika salah satu dari kriteria ini gagal, sistem tetap mempertahankan **SQLite FTS5 BM25 murni** sebagai arsitektur produksi final.
