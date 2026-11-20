# Alur Pengguna (User Journeys) — KerjaiAja
## Pemetaan Pengalaman Pengguna: Dari Onboarding hingga Penyelesaian Soal

Dokumen ini memetakan alur perjalanan pengguna (*user journeys*) untuk interaksi inti KerjaiAja: Onboarding Guest-First, Manajemen Tugas Mode A, Asisten Kerja Mode B, dan Foto Soal & Pembahasan.

---

## 1. Journey 1: Guest-First Onboarding & Pembuatan Tugas Pertama (Mode A)
```mermaid
sequenceDiagram
    autonumber
    actor User as Mahasiswa Baru
    participant App as Aplikasi KerjaiAja
    participant Room as Room DB Lokal

    User->>App: Buka Aplikasi Pertama Kali
    App-->>User: Tampilkan Welcome Screen Neo-Bauhaus (Tanpa Login Paksa)
    User->>App: Ketuk "Mulai Sekarang (Mode Tamu)"
    App->>Room: Inisialisasi User Guest (is_guest = true)
    App-->>User: Dashboard Tugas Bersih (Mode A Aktif)
    User->>App: Tambah Tugas Baru ("Laporan Fisika Dasar", Deadline Besok 17:00)
    App->>Room: INSERT INTO tasks (deterministic_priority = 85)
    App-->>User: Kartu Tugas Muncul dengan Badge CRITICAL (Instan < 5 ms)
```

---

## 2. Journey 2: Alur Foto Soal & Pembahasan (Mode B — Jalur A OCR-First)
```mermaid
sequenceDiagram
    autonumber
    actor User as Pelajar (Mode B Aktif)
    participant UI as Layar Foto Soal
    participant MLKit as Google ML Kit OCR (On-Device)
    participant Engine as Gemma Text Engine (LiteRT-LM)
    participant Validator as AnswerValidator (Deterministik)

    User->>UI: Ketuk Tab "Foto Soal" -> Buka Kamera
    User->>UI: Ambil Foto Soal Matematika (SPLDV / Integral)
    UI-->>User: Layar Crop & Rotate (Sesuaikan Area Soal)
    User->>UI: Ketuk "Proses Soal"
    UI->>MLKit: Ekstraksi Teks Citra On-Device (< 300 ms)
    MLKit-->>UI: Raw Text: "Tentukan himpunan penyelesaian dari 2x + 3y = 12..."
    UI-->>User: Tampilkan Teks OCR untuk Koreksi Manual (Human-in-the-Loop)
    User->>UI: Verifikasi / Perbaiki Angka -> Ketuk "Dapatkan Pembahasan"
    UI->>Engine: Generate Structured JSON (Chat Template + RAG Referensi)
    Engine-->>Validator: Raw JSON Response
    Validator->>Validator: Validasi Aritmetika Deterministik & Konsistensi Langkah
    Validator-->>UI: Validated StructuredSolution (7 Bagian Rapi)
    UI-->>User: Render Pembahasan Interaktif (Konsep, Langkah, Jawaban, Cek Ulang)
    User->>UI: Ketuk "Simpan Pembahasan ke Catatan Belajar"
    UI->>UI: Hapus file cache foto sementara dari memori
```

---

## 3. Journey 3: Kolaborasi Tim & Migrasi Akun Cloud
```mermaid
sequenceDiagram
    autonumber
    actor User as Ketua Kelompok
    participant App as Aplikasi KerjaiAja
    participant Auth as Supabase Auth
    participant Sync as SyncWorker & Supabase PostgREST
    participant RT as Supabase Realtime WebSocket

    User->>App: Ketuk Menu "Buat Tim Kelompok"
    App-->>User: Modal: "Fitur Tim Membutuhkan Akun Cloud untuk Sinkronisasi"
    User->>Auth: Masuk / Daftar via Email & Password
    Auth-->>App: JWT Session Valid (auth.uid = 1234)
    App->>App: Migrasi Transparan: UPDATE tasks SET user_id = '1234'
    App->>Sync: Trigger SyncWorker (Upload Data Historis ke Supabase Postgres)
    User->>App: Buat Tim "Proyek Basis Data" -> Tambah 2 Teman
    App->>RT: Subscribe Channel team_channel_xyz
    Note over App,RT: Teman mengupdate status tugas kelompok -> Layar Ketua langsung terupdate real-time!
```
