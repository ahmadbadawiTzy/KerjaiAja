# RAG Evaluation Plan & Golden Test Suite
## Metodologi Pengujian Akurasi Retrieval, Kualitas Jawaban, dan Tolok Ukur Anti-Halusinasi

- **Status:** APPROVED & LOCKED
- **Lokasi Dokumen:** `docs/testing/RAG_EVALUATION_PLAN.md`
- **Terkait:** [`KNOWLEDGE_INTELLIGENCE_LAYER.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/KNOWLEDGE_INTELLIGENCE_LAYER.md), [`AI_EVALUATION_STRATEGY.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/ai/AI_EVALUATION_STRATEGY.md)

---

## 1. Filosofi Evaluasi RAG

Sistem RAG profesional tidak boleh diklaim berkualitas tinggi hanya karena berhasil mengembalikan beberapa potongan teks. Pengujian harus dilakukan menggunakan **metrik terukur (*verifiable metrics*)**, **golden dataset acuan**, dan **pengujian skenario kegagalan ekstrem (*stress test*)**.

Dua domain dievaluasi secara terpisah:
- **Evaluasi Domain Produktivitas:** Mengukur ketepatan dekomposisi tugas, identifikasi blocker, dan relevansi catatan masa lalu.
- **Evaluasi Domain Akademik:** Mengukur akurasi rumus, kebenaran definisi konsep, kepatuhan sitasi sumber, dan tingkat penolakan halusinasi.

---

## 2. Metrik Pengujian Standar

### 2.1 Metrik Efektivitas Retrieval (Retrieval Metrics)
1. **Precision@K ($K \in \{1, 3, 5\}$):** Proporsi dokumen yang benar-benar relevan dari $K$ dokumen teratas yang dikembalikan sistem. Target: $\text{P@3} \ge 85\%$.
2. **Recall@K ($K \in \{3, 5\}$):** Proporsi dokumen relevan yang berhasil ditemukan dari seluruh dokumen relevan yang ada di database. Target: $\text{Recall@5} \ge 80\%$.
3. **Mean Reciprocal Rank (MRR):** Rata-rata kebalikan dari posisi dokumen relevan pertama ($1 / \text{rank}_i$). Target: $\text{MRR} \ge 0.80$.
4. **Normalized Discounted Cumulative Gain (nDCG@5):** Kualitas urutan peringkat dengan memberikan bobot lebih tinggi pada dokumen relevan di urutan pertama. Target: $\text{nDCG@5} \ge 0.85$.
5. **Retrieval Latency:** Waktu eksekusi kueri FTS5 di Room SQLite. Target: $\le 25\text{ ms}$ pada 95th percentile (P95).

### 2.2 Metrik Kualitas Jawaban AI (Answer Quality Metrics)
1. **Groundedness:** Persentase klaim dalam jawaban yang secara eksplisit didukung oleh teks di dalam potongan referensi lokal. Target: $\ge 92\%$.
2. **Faithfulness:** Ketiadaan kontradiksi antara kesimpulan model dengan fakta yang tertulis pada materi rujukan. Target: $\ge 95\%$.
3. **Citation Correctness:** Validitas sitasi sumber (nama buku, bab, nomor halaman). Dilarang ada sitasi fiktif (*hallucinated citations*). Target: $100\%$ valid terhadap dokumen lokal.
4. **Hallucination Rate:** Proporsi jawaban yang mengandung rumus atau fakta yang salah/dikarang sendiri. Target: $\le 5\%$ (ditekan oleh *Answer Validation Layer*).
5. **Abstention Correctness:** Ketepatan sistem dalam menyatakan *"Referensi materi tidak ditemukan di perangkat"* saat kueri berada di luar basis pengetahuan lokal. Target: $\ge 95\%$.

---

## 3. Matriks 15 Skenario Uji (Test Scenarios)

| No | Skenario Pengujian | Input Kueri / Dokumen | Perilaku yang Diharapkan (*Expected Outcome*) |
| :--- | :--- | :--- | :--- |
| 1 | **Exact Keyword Match** | *"hukum newton 2 bidang miring"* | Mengembalikan bab Hukum Newton urutan teratas (Rank 1). |
| 2 | **Synonym / Parafrase** | *"aturan pergerakan benda miring"* | Normalisasi kueri mengekstrak token gerak dan bidang; BM25 menemukan materi terkait. |
| 3 | **Ambiguous Query** | *"gaya"* (tanpa konteks mapel) | Filter metadata meminta klarifikasi atau menyaring berdasarkan mata pelajaran aktif. |
| 4 | **Multi-Topic Query** | *"perbedaan mitosis dan meiosis pada genetika"* | Mengambil 2 chunk berbeda yang mencakup kedua topik pembelahan sel. |
| 5 | **Large Document Ingestion** | Berkas referensi 120 halaman (12 MB PDF) | Ingestion berhasil memecah menjadi ~250 chunks tanpa OOM pada ponsel 4GB. |
| 6 | **Duplicate Documents** | Mengunggah dokumen kurikulum yang sama 2x | Sistem mendeteksi `content_hash` yang identik dan menolak duplikasi indeks. |
| 7 | **Outdated References** | Dokumen lama berstatus `REVOKED` | Dokumen berstatus revoked secara deterministik dikeluarkan dari hasil kueri. |
| 8 | **Empty Knowledge Base** | Kueri pada mapel yang belum diunduh | Sistem secara transparan menyatakan materi lokal belum diunduh; tidak berhalusinasi. |
| 9 | **Low-Quality OCR Text** | Teks foto soal buram / skor keyakinan 0.45 | Sistem menampilkan peringatan pada UI agar siswa mengoreksi teks sebelum retrieval. |
| 10 | **Unauthorized Document** | Kueri mencoba mengakses tugas milik `user_B` | Otorisasi pra-retrieval SQL memblokir akses; mengembalikan 0 baris (*empty result*). |
| 11 | **Deleted Document Sync** | Dokumen dihapus di CMS | Android delta sync menghapus baris di `academic_chunks` dan membersihkan `academic_fts`. |
| 12 | **Offline Query** | Perangkat dalam Airplane Mode | Pencarian FTS5 lokal dan inferensi model on-device berjalan normal tanpa hambatan. |
| 13 | **Mixed-Language Query** | *"rumus momentum and impulse"* | Tokenizer bilingual memetakan istilah ke bab Momentum dan Impuls Fisika. |
| 14 | **Mathematical Formulas** | Soal integral tentu $\int_0^2 (3x^2 + 2x) dx$ | Parser chunking mempertahankan struktur formula integral utuh tanpa terpotong. |
| 15 | **Table-Based Content** | Tabel periodik unsur / data massa atom | Baris header tabel diikutsertakan di awal chunk sehingga nilai angka memiliki label. |

---

## 4. Struktur Golden Test Set

Dataset evaluasi disimpan dalam format JSON deterministik (`golden_academic_rag_dataset.json`):
```json
[
  {
    "query_id": "GOLDEN_MATH_001",
    "subject": "MATEMATIKA",
    "query_text": "Bagaimana cara menentukan titik puncak fungsi kuadrat f(x) = ax^2 + bx + c?",
    "expected_document_id": "DOC_MATH_KURMER_01",
    "expected_section": "Bab 2: Persamaan dan Fungsi Kuadrat",
    "expected_chunk_ids": ["CHUNK_MATH_042", "CHUNK_MATH_043"],
    "required_formula_snippet": "x_p = -b / (2*a)",
    "expected_behavior": "RETURN_TOP_CHUNKS_AND_FORMULA"
  },
  {
    "query_id": "GOLDEN_OUT_OF_DOMAIN_002",
    "subject": "ASTRONOMI_LANJUT",
    "query_text": "Hitung radius Schwarzschild lubang hitam bermassa 10 massa matahari",
    "expected_behavior": "ABSTAIN_REFERENCE_NOT_FOUND"
  }
]
```
Golden test set ini dijalankan secara otomatis pada CI/CD untuk memastikan tidak ada regresi pada performa mesin retrieval.
