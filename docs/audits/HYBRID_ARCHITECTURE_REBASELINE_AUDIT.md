# AUDIT & REBASELINE ARSITEKTUR HYBRID: KERJAIAJA
## Supabase Cloud Services + Optional On-Device Google Gemma AI

**Tanggal:** 20 November 2026  
**Status:** Disetujui & Diterapkan  
**Ruang Lingkup:** Evaluasi komprehensif transformasi arsitektur dari Pure Local AI / Zero Cloud menjadi Hybrid Offline-First (Supabase BaaS + Optional Local Gemma AI).

---

## 1. Latar Belakang & Motivasi Rebaseline

Pada iterasi sebelumnya, KerjaiAja sempat ditransformasi menjadi arsitektur murni on-device offline tanpa backend cloud apa pun guna menghilangkan beban operasional server inference dan biaya hosting. Namun, pendekatan *pure offline* memunculkan keterbatasan fungsional yang signifikan:
1. **Tidak Adanya Sinkronisasi Lintas Perangkat:** Pelajar dan mahasiswa yang menggunakan lebih dari satu perangkat (misalnya HP Android pribadi dan tablet belajar) tidak dapat mengakses data tugas dan kalender secara konsisten.
2. **Ketiadaan Kolaborasi Tim:** Proyek tugas kelompok yang menjadi kebutuhan vital mahasiswa tidak dapat berjalan secara sinkron tanpa kanal sinkronisasi cloud.
3. **Risiko Kehilangan Data:** Jika perangkat rusak atau hilang, seluruh data akademik lokal pengguna berisiko musnah tanpa adanya cloud backup yang aman.

Untuk mengatasi kelemahan tersebut tanpa mengorbankan prinsip privasi data dan efisiensi biaya nol rupiah, diputuskan arsitektur baru: **Hybrid Offline-First Productivity Platform**.

---

## 2. Pilar Arsitektur Baru

### A. Dual Experience Architecture
- **Mode A (Produktivitas Penuh Tanpa AI):**
  Aplikasi 100% fungsional tanpa AI. Fitur manajemen tugas, jadwal kuliah/sekolah, kalender akademik, beban kerja, deterministik prioritas cerdas, backup cloud Supabase, dan kolaborasi tim tetap berjalan optimal. Pengguna tidak dipaksa mengunduh model AI.
- **Mode B (Produktivitas dengan Asisten Kerja On-Device Gemma Opsional):**
  Bagi pengguna dengan perangkat yang memenuhi syarat hardware dan memilih untuk mengaktifkan AI, model Google Gemma (Gemma 3 1B IT / 270M) dijalankan langsung di perangkat via LiteRT-LM. Fitur Asisten Kerja berfokus pada tugas kontekstual (*Ask About My Work*, *Task Breakdown*, *Smart Planning*, *Contextual Recommendation*).

### B. Supabase Managed BaaS (Official Kotlin Client)
- Menggunakan pustaka resmi [io.github.jan-tennert.supabase:bom:3.9.0](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/gradle/libs.versions.toml) dengan engine Ktor Android.
- Layanan yang dimanfaatkan:
  - **Supabase Auth:** Email/password, Magic Link, dan Google OAuth (opsional).
  - **Supabase PostgreSQL & PostgREST:** Penyimpanan data terstruktur dengan relasi kuat.
  - **Row Level Security (RLS):** Keamanan data multi-tenant di level database Postgres.
  - **Supabase Realtime:** Sinkronisasi instan untuk kolaborasi tugas kelompok via Postgres CDC / Broadcast channels.
  - **Supabase Storage:** Penyimpanan lampiran file tugas dokumen akademik (PDF, docx, gambar) dengan kuota ketat.
- **Batasan Keras:**
  - JANGAN membuat backend server custom (Go, Gin, Node.js, Python server).
  - JANGAN menyimpan model weights AI (Gemma .bin/.litertlm) di Supabase Storage.
  - JANGAN menjalankan cloud inference atau Edge Functions untuk AI.

### C. Zero AI Infrastructure & Zero External AI API
- Tidak ada panggilan API ke Gemini API, OpenAI API, Anthropic Claude, Groq, atau inference provider cloud lainnya.
- Privasi data pengguna 100% terjamin: konten prompt akademik dan catatan tugas lokal tidak pernah dikirim ke AI cloud pihak ketiga.
- Operasional AI berbiaya Rp0 (menggunakan komputasi CPU/NPU lokal perangkat pengguna).

### D. Device Capability Management
Modul [DeviceCapabilityManager.kt](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/app/src/main/java/com/kerjaiaja/app/core/device/DeviceCapabilityManager.kt) mengevaluasi kelayakan hardware sebelum mengizinkan pengunduhan model:
- RAM bebas aktual ($\ge 1.5\text{ GB}$).
- Arsitektur CPU 64-bit (`arm64-v8a` / `x86_64`).
- Ruang penyimpanan internal ($\ge 3.0\text{ GB}$).
- Status termal perangkat dan tingkat baterai ($\ge 30\%$).
- Tingkatan kelayakan: `RECOMMENDED`, `LIMITED`, `NOT_RECOMMENDED`, `UNSUPPORTED`.

### E. Model Download & Storage Lifecycle
- Model weights diunduh dari kanal resmi Google AI Edge / Hugging Face via `DownloadManager` Android dengan resume support dan verifikasi SHA-256.
- Model disimpan di direktori privat aplikasi (`context.noBackupFilesDir/models/`).
- Pengguna memiliki kontrol penuh: toggle on/off, opsi unduh hanya via Wi-Fi, dan tombol hapus model seketika untuk membebaskan penyimpanan.

### F. Asisten Kerja (Task-Contextual Assistant, Non-Generic Chatbot)
- Dilarang membuat antarmuka generic open-ended chatbot.
- Interaksi AI dibatasi pada asisten kontekstual seputar pekerjaan pengguna:
  1. *Ask About My Work* (Tanya beban dan status tugasku).
  2. *Task Breakdown* (Pemecahan tugas kompleks menjadi sub-tugas terukur).
  3. *Smart Planning* (Saran alokasi blok waktu belajar).
  4. *Contextual Recommendations* (Rekomendasi aksi prioritas berdasarkan konteks saat ini).
  5. *Work Review* (Review kesiapan dan kelengkapan tugas sebelum dikumpulkan).

### G. Guest-First & Offline-First Sync Architecture
- Pengguna dapat langsung menggunakan aplikasi tanpa membuat akun (Guest Mode). Data tersimpan lokal di Room Database.
- Saat pengguna memutuskan untuk login/signup Supabase, data lokal secara transparan dimigrasikan ke cloud user ID mereka.
- Sinkronisasi lokal ke cloud menggunakan antrean `sync_pending_queue`, soft deletion dengan `deleted_at` (tombstone), dan strategi resolusi konflik multi-device yang tangguh.

### H. Cost Guardrails: Rp0 dalam Supabase Free Tier
- Database space: Dibatasi maksimal 500 MB (skema relasional dioptimalkan, vacuum berkala).
- Storage space: Dibatasi maksimal 1 GB untuk dokumen akademik terkompresi; batas file upload 5 MB.
- Bandwidth (Egress): Dibatasi $\le 2\text{ GB/bulan}$ via sync diff dan caching lokal agresif.
- Inactivity Pausing Mitigation: Notifikasi ke pengguna atau mekanisme keep-alive ringan yang mematuhi terms Free Plan.

---

## 3. Matriks Perbandingan Arsitektur

| Aspek | Pure Local AI (Iterasi Sebelumnya) | Hybrid Offline-First (Baseline Baru) |
| :--- | :--- | :--- |
| **Penyimpanan Lokal** | Room Database (SQLite) + FTS5 | Room Database (SQLite) + FTS5 |
| **Penyimpanan Cloud** | Tidak ada | Supabase PostgreSQL Managed BaaS |
| **Autentikasi** | Lokal / No-auth | Guest-first + Supabase Auth |
| **Kolaborasi Tim** | P2P Local / Ekspor File Manual | Supabase Realtime + PostgreSQL RLS |
| **Sinkronisasi Lintas Perangkat** | Tidak didukung | Didukung via Supabase Sync Worker |
| **Inference AI** | On-Device Gemma (LiteRT-LM) | On-Device Gemma (LiteRT-LM) |
| **Ketergantungan AI** | Opsional | Opsional (Dual Experience Mode A & B) |
| **Biaya Server / Cloud** | Rp0 | Rp0 (Free Tier Guaranteed Guardrails) |
| **Custom Server Maintenance** | Nol (tidak ada server) | Nol (Managed BaaS, no Go/Gin/Node server) |

---

## 4. Kesimpulan & Status Migrasi

Arsitektur Hybrid Offline-First memberikan keseimbangan terbaik antara:
1. **Keamanan & Privasi:** Data AI tetap 100% lokal di perangkat pengguna.
2. **Fungsionalitas Modern:** Sinkronisasi cloud multi-perangkat dan kolaborasi tugas tim berjalan mulus via Supabase BaaS.
3. **Biaya & Pemeliharaan:** Zero infrastructure maintenance tanpa biaya operasional (Rp0).
4. **Inklusivitas Pengguna:** Pengguna perangkat low-end tetap menikmati aplikasi produktivitas yang super cepat dan lengkap (Mode A).
