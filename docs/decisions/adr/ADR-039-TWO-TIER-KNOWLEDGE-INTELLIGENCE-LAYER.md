# ADR-039: Two-Tier Knowledge Intelligence Layer Architecture (Productivity vs Academic Domains)

## Status
**ACCEPTED & LOCKED** (2026-11-20) — Menetapkan arsitektur pemisahan domain pengetahuan dan mesin retrieval lokal.

---

## 1. Konteks (Context)
Aplikasi KerjaiAja memiliki dua fitur utama yang memanfaatkan kecerdasan buatan on-device: **Asisten Kerja** (pengelolaan produktivitas tugas mahasiswa) dan **Foto Soal & Pembahasan** (pemecahan soal akademik untuk 12 rumpun mata pelajaran).  
Menggabungkan data tugas pribadi mahasiswa dengan materi kurikulum akademik ke dalam satu indeks pencarian tunggal menimbulkan risiko pelanggaran privasi (UU PDP No. 27/2022), pencemaran konteks prompt (*context contamination*), dan pemborosan alokasi memori KV-Cache model AI on-device.

---

## 2. Keputusan (Decision)
1. **Pemisahan Fisik & Logis Dua Domain Pengetahuan:**
   - **Domain A (Productivity Knowledge):** Mengindeks tugas, sub-tugas, proyek, deadline, dan jadwal milik mahasiswa di Room Database (`tasks_fts`). Hanya dapat diakses oleh pemilik data (`owner_id = current_user_id`).
   - **Domain B (Academic Knowledge):** Mengindeks referensi kurikulum, konsep, rumus, dan materi ajar terkurasi untuk 12 rumpun mata pelajaran di tabel virtual terpisah (`academic_fts`). Bersifat publik bagi mahasiswa dan terisolasi dari data tugas personal.
2. **Adopsi SQLite FTS5 BM25 Sebagai Baseline Utama:**
   - Menggunakan modul native SQLite FTS5 dengan fungsi pembobotan BM25 dan pembobotan kolom spesifik (judul bab, kata kunci, isi).
   - Menunda pengadopsian vector database / embedding model lokal sampai pengujian nilai tambah (*Proof-of-Value*) terpenuhi ($\ge 15\%$ peningkatan Recall@5 tanpa lonjakan RAM $>100\text{ MB}$).
3. **100% On-Device & Zero Cloud Transmission:**
   - Seluruh kueri dan ekstraksi pengetahuan dieksekusi luring di perangkat pengguna tanpa transmisi data prompt ke server eksternal.

---

## 3. Konsekuensi (Consequences)
### Positif:
- Menjamin privasi akademik mahasiswa 100% terlindungi (tugas pribadi tidak pernah bocor ke fitur pemecahan soal).
- Latensi kueri sangat rendah ($\le 18\text{ ms}$).
- Penggunaan memori minimal (<15 MB RAM overhead), aman untuk ponsel Android berkapasitas RAM 4 GB.
### Negatif:
- Memerlukan pemeliharaan dua tabel virtual FTS5 dan sekumpulan trigger sinkronisasi otomatis di Room Database.
