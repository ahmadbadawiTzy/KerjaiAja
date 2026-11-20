# ADR-041: Context Engineering, Prompt Injection Defense, and Offline-First Knowledge Sync

## Status
**ACCEPTED & LOCKED** (2026-11-20) — Menetapkan rekayasa konteks, proteksi data pasif, dan sinkronisasi delta luring.

---

## 1. Konteks (Context)
Dokumen referensi akademik yang diunggah pengguna atau diperoleh dari sumber publik dapat menjadi celah serangan *prompt injection* jika model AI memperlakukan teks referensi sebagai instruksi sistem. Selain itu, sinkronisasi seluruh basis pengetahuan kurikulum secara massal berpotensi menghabiskan kuota seluler pengguna dan kuota egress Supabase Free Tier ($<2$ GB/bln).

---

## 2. Keputusan (Decision)
1. **Pemisahan Tegas Data vs Instruksi & Prompt Injection Defense:**
   - Seluruh teks hasil retrieval disuntikkan ke dalam prompt sebagai **DATA PASIF** di dalam tag pembatas netral (`<retrieved_references>`).
   - Melakukan *delimiter sanitization* (meng-*escape* karakter `<`, `>`, dan backtick ganda).
   - Menginstruksikan system prompt secara tegas untuk mengabaikan perintah operasional apa pun yang tertulis di dalam blok data referensi.
2. **Feature-Dedicated Context Budgeting:**
   - *Productivity Context Builder:* Maksimal 800 tokens (informasi tugas aktif, blocker langsung, kapasitas harian).
   - *Academic Context Builder:* Maksimal 1200 tokens (teks soal terverifikasi, rumus baku, 3–5 chunks terbaik).
3. **Offline-First Delta Knowledge Sync & Kontrol Pengguna:**
   - Klien Android menyinkronkan materi via `updated_at > :lastSyncTime` hanya untuk materi berstatus `is_published = true`.
   - Mahasiswa memegang kendali penuh atas pemilihan rumpun mata pelajaran yang diunduh ke ponsel (kuota maksimal 100 MB).
   - Menggunakan deletion marker untuk mencabut materi usang dari indeks FTS5 lokal secara atomik.

---

## 3. Konsekuensi (Consequences)
### Positif:
- Sistem kebal terhadap manipulasi prompt injection dari dokumen teks.
- Anggaran token terkendali ketat; risiko crash OOM pada model LiteRT-LM diminimalkan.
- Penggunaan kuota internet mahasiswa sangat hemat karena hanya data perubahan (*delta*) yang ditransmisikan.
### Negatif:
- Kueri yang membutuhkan referensi sangat panjang terpaksa mengalami pemotongan (*truncation*) pada chunk berbobot BM25 terendah.
