# ADR-015: In-App Local RAG Menggunakan SQLite FTS5 BM25

## Status
**ACCEPTED** (2026-11-20) — Menetapkan arsitektur RAG lokal untuk rilis MVP.

---

## 1. Konteks (Context)
Untuk menghasilkan dekomposisi tugas dan analisis tenggat waktu yang kontekstual, model LLM on-device membutuhkan informasi terkait: mata kuliah, beban SKS, riwayat tugas serupa, dan sisa waktu belajar. Mengirim seluruh basis data ke dalam prompt dilarang karena keterbatasan *context window* (anggaran maksimal ~1024–1536 tokens) dan risiko memicu *Out-Of-Memory* (OOM).

---

## 2. Keputusan (Decision)
1. **Adopsi SQLite FTS5 (Lexical Retrieval BM25):**  
   KerjaiAja mengimplementasikan tabel virtual **SQLite FTS5** (`tasks_fts`) di dalam database Room lokal. FTS5 menyediakan algoritma pencarian teks penuh terindeks dengan fungsi ranking relevansi bawaan BM25 (*Best Matching 25*).
2. **Zero Additional Model Overhead:**  
   Pencarian berbasis FTS5 tidak membutuhkan file model embedding tambahan dan tidak membebani alokasi RAM ekstra.
3. **Penyaringan Konteks Bertingkat (Selective Context Pruning):**  
   - Mengambil maksimal 3–5 tugas paling relevan yang memiliki korelasi mata kuliah atau tag yang sama.
   - Mengabaikan tugas yang telah selesai lebih dari 30 hari yang lalu.
   - Membersihkan nomor kontak atau informasi pribadi sebelum membangun prompt.
4. **Evaluasi Semantic Retrieval (EmbeddingGemma 2) Ditunda ke Pasca-MVP:**  
   Model embedding vektor lokal ditangguhkan sampai keefektifan FTS5 divalidasi pada pengujian pengguna nyata.

---

## 3. Konsekuensi (Consequences)
### Positif:
- Kueri pencarian teks lokal sangat cepat ($\le 25\text{ ms}$).
- Tidak ada pengeluaran memori untuk runtime inferensi embedding.
- Integritas indeks FTS5 terjaga secara otomatis melalui SQLite triggers.
### Negatif:
- Pencarian leksikal sensitif terhadap variasi sinonim kata (misal: "laporan" vs "makalah") dibandingkan pencarian berbasis semantik dense vector.
