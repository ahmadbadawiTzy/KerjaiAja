# Spesifikasi Kebutuhan Produk (Product Requirements) — KerjaiAja

Dokumen ini mendefinisikan kebutuhan fungsional dan non-fungsional aplikasi **KerjaiAja** pasca-rebaseline arsitektur hybrid (**Hybrid Offline-First: Supabase Cloud Services + Optional On-Device Google Gemma AI**).

---

## 1. Tiga Pilar Inti Produk (Core Pillars)

KerjaiAja dirancang dengan tiga pilar produk yang seimbang dan beroperasi secara hybrid offline-first:
1. **Personal Academic Productivity (Pilar 1)**
2. **Intelligent & Explainable Task Prioritization via Deterministic Engine & Optional On-Device AI (Pilar 2)**
3. **Team Collaboration & Group Accountability via Supabase Realtime & RLS (Pilar 3)**

---

## 2. Kebutuhan Fungsional (Functional Requirements)

### PILAR 1: PERSONAL ACADEMIC PRODUCTIVITY

#### FR-01: Manajemen Tugas Akademik (Academic Task Management)
- Sistem menyediakan kemampuan CRUD tugas akademik secara cepat langsung ke Room Database (Local-Write-First).
- Atribut tugas mencakup: judul, deskripsi markdown, mata pelajaran (*subject_id*), deadline, estimasi durasi menit, bobot nilai persentase, tingkat kesulitan (1–5), kategori penugasan (individu/kelompok), status (`TODO`, `IN_PROGRESS`, `IN_REVIEW`, `COMPLETED`, `OVERDUE`), deterministik prioritas score (0–100), dan nilai override pengguna.
- Tersinkronisasi ke cloud Supabase PostgreSQL jika pengguna login, dengan antrean mutasi offline `sync_pending_queue`.

#### FR-02: Manajemen Mata Kuliah & Jadwal (Subject & Schedule Management)
- Pengelompokan tugas berdasarkan mata kuliah atau blok semester.
- Informasi mata kuliah mencakup: nama mata kuliah, kode, nama pengajar, jadwal pertemuan mingguan, lokasi, dan tag warna fungsional.
- Indikator beban akademik visual untuk mendeteksi penumpukan tugas sebelum masa ujian.

#### FR-03: Dekonstruksi Tugas Berjenjang (Hierarchical Assignment Breakdown)
- Kemampuan memecah tugas kompleks menjadi sub-tugas berjenjang (*actionable subtasks/milestones*).
- Setiap sub-tugas memiliki estimasi waktu pengerjaan, status ketercapaian, dan urutan urut.
- Visualisasi persentase progres dinamis berbasis sub-tugas yang telah ditandai selesai.

#### FR-04: Sesi Fokus & Manajemen Waktu (Study Planning & Focus Timer)
- Pengatur waktu fokus berbasis teknik Pomodoro adaptif yang terikat langsung ke sub-tugas aktif.
- Mode anti-distraksi sederhana pada tampilan Compose saat timer berjalan.
- Pencatatan otomatis waktu riil yang dihabiskan (*actual time spent*) untuk membandingkan estimasi vs kenyataan.

#### FR-05: Daftar Periksa Pengumpulan (Pre-Flight Submission Checklist)
- Verifikasi persyaratan sebelum tugas dikumpulkan ke portal akademik:
  - Format berkas (PDF, DOCX, ZIP, link Git repo).
  - Standar penamaan berkas sesuai format penugasan.
  - Lampiran dokumen akademik (disimpan lokal atau diunggah ke Supabase Storage maks 5 MB jika login).
  - Pencatatan timestamp bukti penyerahan tugas.

---

### PILAR 2: INTELLIGENT & EXPLAINABLE TASK PRIORITIZATION

#### FR-06: Perhitungan Prioritas Deterministik (Deterministic Priority Engine)
- Menghitung skor prioritas dasar (0–100) secara deterministik dan instan di ponsel tanpa bergantung pada jaringan atau AI:
  - *Time to Deadline:* Skala urgensi bertingkat; tugas overdue otomatis mendapat base score 85+.
  - *Academic Weight:* Kontribusi bobot nilai terhadap nilai akhir.
  - *Difficulty & Duration:* Rasio waktu yang tersisa terhadap estimasi durasi kerja.
- 4 Kategori Urgensi:
  - **CRITICAL (Skor 80–100):** Wajib dikerjakan segera (deadline <24 jam atau overdue).
  - **HIGH (Skor 60–79):** Memerlukan progres segera (deadline 2–3 hari).
  - **MEDIUM (Skor 40–59):** Tugas terjadwal normal (deadline 4–7 hari ke depan).
  - **LOW (Skor 0–39):** Tugas bacaan mandiri atau deadline masih panjang (>1 minggu).

#### FR-07: Penalaran Kontekstual Asisten Kerja On-Device (Gemma Reasoning & Local RAG)
- Engine inferensi on-device (Google Gemma 3 1B IT / 270M via LiteRT-LM) menganalisis konteks tugas lokal via SQLite FTS5 BM25 untuk mengidentifikasi risiko tersembunyi.
- Memberikan penjelasan transparan (*explainable rationale*): *"Diprioritaskan ke HIGH karena memerlukan riset literatur dan survei lapangan sebelum analisis data."*
- Jika model belum diunduh atau spesifikasi hardware tidak mencukupi, sistem tetap menyajikan skor deterministik tanpa error (*graceful fallback*).

#### FR-08: Kendali Penuh Pengguna & Anti-Guilt UX
- Pengguna berhak mengubah (*override*) prioritas yang disarankan oleh sistem kapan saja.
- Jika tugas melewati tenggat (*overdue*), sistem tidak menampilkan pesan yang memojokkan, melainkan menyediakan opsi adaptif: *Reschedule with Assistant*, *Split into 15-Min Steps*, atau *Mark Completed*.

---

### PILAR 3: TEAM COLLABORATION & CLOUD SERVICES

#### FR-09: Ruang Kerja Tim Kolaboratif (Team Workspaces)
- Pembuatan proyek kelompok akademik dengan manajemen anggota dan peran (`owner`, `admin`, `member`).
- Pembagian tugas kelompok (*team tasks*), penetapan penanggung jawab (*assignee*), dan pelacakan status progres (`todo`, `in_progress`, `review`, `done`).
- Proteksi akses data multi-tenant via PostgreSQL Row Level Security (RLS) di Supabase.

#### FR-10: Sinkronisasi Real-Time & Realtime Presence
- Anggota tim menerima pembaruan status tugas kelompok secara instan melalui Supabase Realtime WebSocket (Postgres CDC).
- Otomatis memutus koneksi WebSocket saat pengguna keluar dari layar tim untuk menghemat kuota dan baterai ponsel.

#### FR-11: Autentikasi Tamu & Cloud Opsional (Guest-First & Seamless Migration)
- Pengguna dapat langsung menggunakan seluruh fitur pribadi tanpa login (Guest Mode).
- Pengguna dapat mendaftar/login kapan saja via Supabase Auth (Email / Magic Link).
- Seluruh data lokal Guest otomatis dimigrasikan ke user ID Supabase baru secara transparan tanpa kehilangan data.

---

### FITUR KECERDASAN BUATAN: ASISTEN KERJA ON-DEVICE (BUKAN CHATBOT)

#### FR-12: Asisten Kerja Kontekstual Akademik
- Dilarang membuat generic chatbot atau obrolan santai tanpa batas.
- Interaksi dibatasi pada 5 skenario kerja:
  1. *Ask About My Work:* Tanya status beban tugas terdekat.
  2. *Task Breakdown:* Pemecahan otomatis tugas besar menjadi sub-tugas JSON terstruktur.
  3. *Smart Planning:* Saran alokasi blok waktu belajar yang realistis.
  4. *Contextual Recommendations:* Rekomendasi aksi tugas berikutnya berdasarkan deterministik score.
  5. *Work Review:* Pemeriksaan kesiapan checklist sebelum pengumpulan tugas.
- Seluruh inferensi berjalan lokal di perangkat pengguna.

#### FR-13: Fitur Foto Soal & Pembahasan (Photo Question Solver)
- Pemecahan soal akademik melalui input foto dari Kamera (CameraX) atau Galeri (Photo Picker).
- Fitur crop, rotate, dan retake sebelum ekstraksi teks.
- Ekstraksi teks lokal via Google ML Kit Text Recognition v2 dengan confidence score per kata.
- Layar peninjauan mandatori (*Human-in-the-Loop Review*) dengan toolbar simbol akademik (pecahan, akar, pangkat, simbol sains/Yunani).
- Pemilihan konteks 12 Rumpun Mata Pelajaran (Matematika, Fisika, Kimia, Biologi, Bhs Indonesia, Bhs Inggris, Sejarah, Geografi, Ekonomi, Sosiologi, PPKn, Lainnya).
- Pembahasan terstruktur 7 bagian: (1) Identifikasi Soal, (2) Informasi yang Diketahui, (3) Yang Ditanyakan, (4) Konsep/Rumus yang Digunakan, (5) Langkah Pengerjaan, (6) Jawaban Akhir, (7) Pemeriksaan & Verifikasi.
- Answer Validation Layer deterministik untuk verifikasi kalkulasi aritmetika, konsistensi satuan, dan uncertainty flagging.
- Pembersihan citra efemeral seketika dari `context.cacheDir` setelah pemrosesan; zero upload ke cloud.

---

## 3. Kebutuhan Non-Fungsional (Non-Functional Requirements)

### NFR-01: Arsitektur Platform Android
- Klien Android Native murni (Kotlin 2.0+, Jetpack Compose, Material 3 Neo-Bauhaus, Room 2.6.1, DataStore, Coroutines & Flow).
- Clean Architecture (Presentation, Domain, Data) dengan pola MVVM dan UDF.
### PILAR 4: WEB ADMIN CMS & OPERATIONAL GOVERNANCE

#### FR-13: Manajemen Kurikulum & Referensi Akademik (Academic Curriculum Management)
- Admin dapat mengelola master 12 rumpun mata pelajaran, ringkasan konsep kurikulum, dan formula kunci di Web Admin CMS.
- Perubahan silabus disinkronkan secara transparan ke aplikasi Android via Delta Sync `academic_references`.

#### FR-14: Sistem Dukungan & Penanganan Keluhan (Feedback & Support System)
- Siswa Android dapat mengirimkan tiket feedback, laporan kesalahan OCR, atau permintaan fitur langsung dari aplikasi.
- Staf Admin dan Support dapat meninjau, mengelompokkan kategori, dan memperbarui status tiket (`NEW`, `IN_REVIEW`, `RESOLVED`, `DISMISSED`) di Web CMS.

#### FR-15: Pengawasan Operasional & Audit Trail Administratif
- Paginasi server-side dan filter pencarian pengguna untuk peninjauan status akun (Aktif/Suspended).
- Pencatatan jejak audit otomatis pada tabel `admin_audit_logs` untuk setiap tindakan mutasi data oleh administrator.
- Proteksi privasi mutlak: seluruh data catatan tugas personal mahasiswa dilarang dibaca oleh staf admin (anti-surveillance).

---

## 3. Kebutuhan Non-Fungsional (Non-Functional Requirements)

### NFR-01: Performa UI & Responsivitas Offline
- Operasi CRUD lokal di Android selesai dalam $< 5\text{ ms}$ (Local-First Room).
- Cold start aplikasi < 800 ms pada perangkat mid-range; frame rate stabil 60/120 FPS.
- First Contentful Paint (FCP) Web Admin CMS $\le 1.2\text{ detik}$ di peramban desktop.

### NFR-02: Zero AI Server & Zero External AI API
- Tidak ada backend server custom (Go/Gin/Node/Python ditiadakan).
- Supabase hanya digunakan sebagai Managed BaaS (Auth, Postgres, Realtime, Storage).
- **Zero Cloud AI API:** Dilarang melakukan panggilan jaringan ke Gemini API, OpenAI API, Anthropic API, Groq, atau inference cloud lainnya.

### NFR-03: Kualifikasi Perangkat Multi-Faktor (Text vs Vision Capability)
- Modul `DeviceCapabilityManager` mengevaluasi kapabilitas secara terpisah:
  - `TextCapability`: Untuk Jalur A (ML Kit OCR + Gemma 3 1B/270M Text). Membutuhkan RAM $\ge 4\text{ GB}$ (RAM bebas $\ge 1.5\text{ GB}$), ABI 64-bit, storage $\ge 3\text{ GB}$.
  - `VisionCapability`: Untuk Jalur B Eksperimental (Gemma 3n E2B Multimodal). Membutuhkan RAM $\ge 8\text{ GB}$ (RAM bebas $\ge 3.0\text{ GB}$), NPU $\ge 15\text{ TOPS}$, storage $\ge 5\text{ GB}$.
- Menghasilkan tingkatan kualifikasi: `RECOMMENDED`, `LIMITED`, `NOT_RECOMMENDED`, `UNSUPPORTED`.
- Transparansi kondisi hardware disajikan secara jelas kepada pengguna sebelum mengunduh model.

### NFR-04: Manajemen Model Download & Storage Lifecycle
- Model weights Google Gemma diunduh terpisah dari APK via Android `DownloadManager` dengan verifikasi SHA-256 dan kemampuan resume.
- Model disimpan di `context.noBackupFilesDir/models/` dengan izin privat.
- Pengguna memiliki tombol hapus model instan di menu Pengaturan.

### NFR-05: Kepatuhan Guardrails Biaya Rp0 Supabase Free Tier & Vercel Hobby
- Database space $\le 500\text{ MB}$, File storage $\le 1\text{ GB}$, Egress bandwidth $\le 2\text{ GB/bulan}$.
- Dilarang menyimpan file model AI di Supabase Storage.
- Menggunakan sinkronisasi delta incremental untuk meminimalkan egress.
- Web Admin CMS mematuhi lisensi non-komersial Vercel Hobby untuk tahap pengujian dan riset.

### NFR-06: Kesiapan Google Play Store & Keamanan
- `compileSdk = 35`, `targetSdk = 35`, `minSdk = 26` (Android 8.0 Oreo).
- Fitur penghapusan akun mandiri (*Account Deletion*) dengan cascading delete di Supabase PostgreSQL.
- Anon key dienkripsi di binary; token sesi disimpan di Android Keystore (AES-256-GCM).
- Zero service role key di browser klien Web CMS.

