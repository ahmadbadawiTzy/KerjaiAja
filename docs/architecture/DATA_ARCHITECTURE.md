# Arsitektur Data (Data Architecture) — KerjaiAja

Dokumen ini menjelaskan perancangan data relasional lokal (Room), pencarian teks penuh (SQLite FTS), penyimpanan preferensi (DataStore), serta strategi sinkronisasi opsional dengan Supabase.

---

## 1. Skema Basis Data Lokal (Room Database)

Basis data lokal menggunakan SQLite yang dibungkus oleh **Room Persistence Library**.

### 1.1 Entitas Inti (Core Entities)

#### 1. `subjects` (Mata Kuliah / Pelajaran)
- `id`: UUID (Primary Key, String)
- `code`: String (Kode MK, misal: "IF2104")
- `name`: String (Nama Mata Kuliah)
- `lecturer_name`: String (Nama Dosen/Pengampu)
- `color_hex`: String (Kode warna penanda tampilan)
- `schedule_day`: Int (Hari pertemuan mingguan, 1-7)
- `schedule_time`: String (Waktu pertemuan)
- `classroom_or_link`: String (Ruang kelas fisik / link meeting daring)
- `created_at`: Long (Timestamp epoch ms)
- `updated_at`: Long (Timestamp epoch ms)

#### 2. `tasks` (Tugas Akademik)
- `id`: UUID (Primary Key, String)
- `subject_id`: UUID (Foreign Key ke `subjects.id`, OnDelete CASCADE)
- `title`: String (Judul tugas)
- `description`: String (Deskripsi/instruksi tugas)
- `due_date`: Long (Timestamp batas pengumpulan)
- `priority`: String (LOW, MEDIUM, HIGH, URGENT)
- `status`: String (TODO, IN_PROGRESS, IN_REVIEW, DONE)
- `task_type`: String (INDIVIDUAL, GROUP, EXAM_PREP)
- `estimated_minutes`: Int (Estimasi durasi pengerjaan)
- `is_ai_assisted`: Boolean (Menandai apakah tugas di-breakdown via AI)
- `created_at`: Long (Timestamp)
- `updated_at`: Long (Timestamp)

#### 3. `subtasks` (Sub-langkah Hasil Breakdown)
- `id`: UUID (Primary Key, String)
- `task_id`: UUID (Foreign Key ke `tasks.id`, OnDelete CASCADE)
- `order_index`: Int (Urutan pengerjaan sub-tugas)
- `title`: String (Aksi konkret sub-tugas)
- `assigned_to`: String? (Nama anggota kelompok jika tugas kelompok)
- `is_completed`: Boolean (Status selesai)
- `completed_at`: Long? (Waktu penyelesaian)

#### 4. `submission_checklists` (Daftar Periksa Pengumpulan)
- `id`: UUID (Primary Key, String)
- `task_id`: UUID (Foreign Key ke `tasks.id`, OnDelete CASCADE)
- `item_text`: String (Kriteria periksa, misal: "Format PDF < 5MB")
- `is_checked`: Boolean (Status validasi pengguna)
- `category`: String (FORMAT, CONTENT, ATTACHMENT, METADATA)

#### 5. `academic_notes` (Catatan Konteks & Silabus)
- `id`: UUID (Primary Key, String)
- `subject_id`: UUID (Foreign Key ke `subjects.id`, OnDelete CASCADE)
- `title`: String (Judul catatan/materi)
- `content`: String (Teks catatan materi atau ringkasan silabus)
- `created_at`: Long
- `updated_at`: Long

---

## 2. SQLite Full-Text Search (FTS4/FTS5)

Untuk mendukung kemampuan pencarian berkecepatan tinggi dan penyusunan konteks RAG lokal (*Local Retrieval-Augmented Generation*), Room dikonfigurasi dengan tabel virtual FTS:

- **Tabel Virtual:** `academic_notes_fts`
- **Tujuan:** Mengindeks seluruh teks pada `academic_notes` dan deskripsi `tasks`.
- **Penggunaan:** Ketika pengguna meminta asisten AI memecah tugas atau merangkum materi, query FTS dijalankan untuk mengambil potongan paragraf materi yang paling relevan dengan instruksi tugas dalam hitungan milidetik (*sub-millisecond retrieval*), yang kemudian diinjeksikan ke dalam prompt konteks lokal.

---

## 3. DataStore Preferences (Konfigurasi & AI State)

Pengaturan non-relasional dan metadata sistem disimpan dalam **Jetpack DataStore Preferences**:

- `ai_model_status`: Status ketersediaan model lokal (`NOT_DOWNLOADED`, `DOWNLOADING`, `VERIFYING`, `READY`, `ERROR`).
- `ai_model_sha256`: Hash integritas model yang terpasang.
- `ai_model_path`: Jalur berkas biner model di internal storage.
- `user_theme_mode`: Preferensi tema tampilan (System, Light, Dark).
- `supabase_sync_enabled`: Status apakah pengguna mengaktifkan sinkronisasi cloud.
- `last_sync_timestamp`: Waktu terakhir sinkronisasi sukses.

---

## 4. Penataan Sinkronisasi Supabase (Opsional)

Jika pengguna mengaktifkan fitur pencadangan cloud:
- Skema PostgreSQL di Supabase merefleksikan skema Room dengan skema UUID yang identik.
- Pengguna diotentikasi via **Supabase Auth** (email atau anonymous).
- **Row-Level Security (RLS)** diterapkan ketat: pengguna hanya dapat membaca dan menulis data yang memiliki `user_id = auth.uid()`.
- Konflik sinkronisasi (*conflict resolution*) diselesaikan dengan strategi *Last-Write-Wins* berbasis `updated_at` atau *Client-Wins* untuk menjaga integritas data lokal pengguna.
