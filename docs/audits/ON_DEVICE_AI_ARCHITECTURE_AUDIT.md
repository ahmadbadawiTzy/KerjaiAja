# Laporan Audit Arsitektur: Migrasi Fundamental Menuju On-Device AI Google Gemma & Local-First — KerjaiAja

- **Tanggal Audit:** 2026-11-20
- **Status Baseline:** REBASELINED & APPROVED
- **Otoritas:** Tim Arsitektur KerjaiAja (Disetujui Pengguna)
- **Dokumen Referensi:** [ADR-014](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-014-ON-DEVICE-GEMMA-LITERTLM.md), [ADR-015](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-015-LOCAL-RAG-SQLITE-FTS5.md), [ADR-016](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-016-REMOVAL-OF-GO-GIN-BACKEND.md), [ADR-017](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-017-OFFLINE-FIRST-DATA-ARCHITECTURE-SUPABASE-DESCOPING.md), [ADR-018](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-018-LOCAL-TEAM-WORKSPACE-PHASED-COLLABORATION.md)

---

## 1. Eksekutif Ringkasan (Executive Summary)

Audit ini dilakukan sebagai respons atas keputusan strategis untuk melakukan **Fundamental Architecture Rebaseline** pada platform KerjaiAja. Keputusan ini secara sadar mengubah paradigma sistem dari model hybrid client-server (Go backend + Supabase managed DB + self-hosted VPS AI `llama.cpp`) menjadi arsitektur **100% Android Native Murni, Offline-First, Local-First, Zero Backend, dan Zero AI Infrastructure** berbasis **Google Gemma On-Device (Gemma 3 1B / Gemma 4 E2B via LiteRT-LM)** dan **Local RAG (SQLite FTS5)**.

Target operasional infrastruktur cloud dan inferensi AI KerjaiAja kini dipatok pada **Rp 0 / $0**, menghilangkan seluruh biaya sewa VPS, lisensi server, dan risiko tagihan token API eksternal pihak ketiga.

---

## 2. Kondisi Arsitektur Sebelum vs Sesudah Rebaseline

| Dimensi Arsitektur | Kondisi Sebelumnya (Hybrid Monolith) | Kondisi Baru (On-Device & Local-First) | Rationale Rekayasa |
| :--- | :--- | :--- | :--- |
| **Pusat Eksekusi AI** | Remote VPS Linux via daemon `llama.cpp` (:8081) | **On-Device Android** via **LiteRT-LM runtime** | Kedaulatan data mutlak, privasi offline 100%, zero network latency |
| **Model AI Utama** | Qwen3-1.7B / Qwen3-4B GGUF Q4_K_M | **Google Gemma 3 1B / Gemma 4 E2B** | Bobot resmi Google teroptimasi untuk mobile via AI Edge |
| **Infrastruktur Backend** | Go + Gin Modular Monolith (`backend/`) | **DIHAPUS (Zero Backend)** | Tidak membutuhkan server transaksi terpusat untuk fungsi produktivitas inti |
| **Database & Auth** | Supabase Managed Cloud (PostgreSQL + RLS + JWT) | **DI-DESCOPE dari MVP (Room + DataStore Lokal)** | Menghilangkan ketergantungan koneksi internet dan cloud identity provider |
| **Arsitektur RAG** | Remote query & backend retrieval | **Local RAG via SQLite FTS5 (BM25)** | Pencarian leksikal cepat langsung di database internal ponsel tanpa overhead jaringan |
| **Distribusi Model AI** | Disimpan di server internal VPS | **On-Demand Download** via persetujuan pengguna | Ukuran APK tetap ramping (<20 MB); bobot model disimpan di app-specific storage |
| **Fitur Interaksi AI** | Asistensi inferensi server | **Non-Chat Contextual Intelligence** | Dilarang membuat chatbot/chat bubble; AI hanya hadir kontekstual pada tugas |
| **Biaya Operasional AI & Server** | ~$5 – $24 / bulan (sewa VPS) | **Rp 0 / $0 (Nol Biaya Tetap)** | Pengguna menjalankan inferensi pada perangkat miliknya sendiri |

---

## 3. Komponen yang Dihapus (To Be Deleted / Removed)

Komponen-komponen berikut diidentifikasi sudah tidak relevan dengan arsitektur baru dan wajib dihapus secara bersih:

1. **Folder Scaffold Backend Go (`backend/`):**
   - `backend/cmd/server/main.go` & `main_test.go`
   - `backend/internal/config/config.go`
   - `backend/go.mod`
   - `backend/.env.example`
   - *Verifikasi Ketergantungan:* Tidak ada kode klien Android yang mengimpor atau terikat dengan direktori `backend/`. Penghapusan berisiko nol terhadap integritas modul Android.
2. **Konfigurasi Server & Deployment:**
   - Bagian container `caddy`, `backend`, dan `ai-service` (`docker-compose.yml`) di `DEPLOYMENT_ARCHITECTURE.md`.
   - Konfigurasi systemd daemon dan alokasi vCPU/RAM server VPS.
3. **Ketergantungan Supabase Cloud:**
   - Library Supabase Go Client, JWT verification middleware, dan konfigurasi skema RLS Supabase cloud dari scope rilis MVP v1.0.
4. **Instruksi Lingkungan Go & Server Runtime:**
   - Petunjuk setup toolchain Go di dokumentasi onboarding pengembangan.

---

## 4. Komponen yang Dipertahankan (To Be Preserved)

Komponen-komponen berikut terbukti solid, stabil, dan tetap dipertahankan sepenuhnya:

1. **Client Android Native:**
   - Bahasa: **Kotlin 2.0+**
   - UI Framework: **Jetpack Compose + Material 3 (Neo-Bauhaus aesthetic)**
   - Pola Arsitektur: **Clean Architecture (Domain, Data, UI) + MVVM + UDF (Unidirectional Data Flow)**
   - Reactive Stream: **Kotlin Coroutines & Flow**
2. **Local Persistence:**
   - **Room Database 2.6.1** untuk entitas relasional terstruktur (Tasks, Subtasks, Subjects, Schedules, TeamProjects).
   - **DataStore Preferences** untuk preferensi aplikasi, status model AI, dan onboarding.
   - **Android Keystore** untuk enkripsi lokal data sensitif.
3. **Deterministic Prioritization Engine:**
   - Algoritma deterministik penentuan skor prioritas (0–100) berbasis tenggat waktu (*deadline weight*), tingkat kesulitan, dan status keterlambatan tetap menjadi sumber kebenaran numerik utama tanpa bergantung pada LLM.
4. **Desain Sistem Neo-Bauhaus:**
   - Tipografi Inter, kontras warna tegas, grid proporsional, dan *anti-guilt UX* tetap menjadi identitas visual KerjaiAja.
5. **Standar Kepatuhan Google Play Store:**
   - Target SDK 35, privasi data lokal tanpa transfer pihak ketiga, dan izin perangkat minimal.

---

## 5. Komponen yang Dirancang Ulang (To Be Redesigned)

1. **Engine Inferensi AI (Dari Remote Llama-Server ke Local LiteRT-LM):**
   - Mengganti HTTP client remote dengan integrasi SDK resmi **LiteRT-LM (Google AI Edge)** di Android.
   - Menyediakan antarmuka modular `AiEngine` dengan implementasi `LiteRtLmEngine` dan `FakeAiEngine` untuk unit testing.
2. **Sistem Local RAG (Dari Konsep Backend ke In-App SQLite FTS5):**
   - Menambahkan tabel virtual FTS5 di Room (`tasks_fts`) untuk pengindeksan leksikal judul tugas, deskripsi, mata pelajaran, dan sub-tugas.
   - Membangun `LocalRagPipeline` yang mengekstraksi konteks terdekat (BM25 ranking), memotong context window sesuai kapasitas token, dan menginjeksinya ke prompt template Gemma.
3. **Alur Manajemen Bobot Model (On-Demand Model Download Manager):**
   - Model **TIDAK DITANAM** di APK. APK didistribusikan ramping (~12–18 MB).
   - Pengguna mengunduh bobot Gemma secara sadar melalui antarmuka pengaturan dengan konfirmasi eksplisit, pengecekan ruang disk bebas ($\ge 3\text{ GB}$), validasi checksum SHA-256, dan opsi penghapusan model sewaktu-waktu.
4. **Kolaborasi Tim (Team Collaboration):**
   - **Kenyataan Teknis:** Tanpa backend cloud terpusat dan tanpa Supabase Realtime, kolaborasi multi-perangkat instan secara online tidak dapat berjalan pada MVP v1.0.
   - **Solusi Rekayasa Jujur:** 
     - **MVP v1.0:** Mendukung **Local Team Workspaces** (pengelolaan tugas kelompok bersama pada satu perangkat) dan **File-Based Project Sharing** (ekspor/impor file `.kerjai` atau JSON terenkripsi via Android Share Sheet).
     - **Pasca-MVP (Roadmap):** Sinkronisasi cloud multi-perangkat (CRDT / P2P sync / optional lightweight sync relay) dijadwalkan secara terpisah tanpa mengorbankan sifat offline-first aplikasi.

---

## 6. Dampak Terhadap Fitur MVP (Feature Scope Impact)

| Fitur / Modul | Status Baru | Dampak Teknis & Pengalaman Pengguna |
| :--- | :---: | :--- |
| **Personal Task CRUD & Tagging** | ✅ **Tersedia Penuh** | Berjalan 100% lokal via Room Database tanpa butuh internet. |
| **Deterministic Priority Engine** | ✅ **Tersedia Penuh** | Kalkulasi prioritas 0–100 dan status urgensi instan di perangkat. |
| **Intelligent Task Breakdown (AI)** | ✅ **Tersedia Lokal** | Berjalan via on-device Gemma. Jika model belum diunduh, fallback ke template silabus manual. |
| **Contextual Task Prioritization (AI)** | ✅ **Tersedia Lokal** | AI memberikan narasi kontekstual penguat skor deterministik secara lokal. |
| **Deadline Risk Analysis (AI)** | ✅ **Tersedia Lokal** | Analisis beban kerja dan risiko keterlambatan langsung dari database Room. |
| **Local RAG Knowledge Injection** | ✅ **Tersedia Lokal** | Ekstraksi konteks tugas relevan via SQLite FTS5 langsung di ponsel. |
| **Team Workspaces (Offline-First)** | 🔄 **Dirancang Ulang** | Pengelolaan tugas kelompok lokal pada perangkat aktif + ekspor/impor berkas tugas tim. |
| **Realtime Cloud Sync & Auth** | ⏸️ **Ditunda ke Pasca-MVP** | Dihapus dari MVP v1.0 karena tidak ada backend server; tidak ada login cloud wajib. |
| **AI Chat / Chatbot Screen** | ❌ **DILICENSEKAN DILARANG** | Aplikasi tidak memiliki fitur percakapan umum; AI hanya melayani aksi produktivitas terfokus. |

---

## 7. Analisis Risiko & Mitigasi

| Risiko Rekayasa | Tingkat Keparahan | Strategi Mitigasi Proaktif |
| :--- | :---: | :--- |
| **Risiko OOM / Low Memory Killer (LMK)** | **TINGGI** | Implementasi `DeviceCapabilityDetector` (ADR-001/014). Model hanya diaktifkan jika RAM bebas $\ge 1.5\text{ GB}$. Cleanup memori agresif sesaat setelah inferensi selesai. |
| **Thermal Throttling & Baterai** | SEDANG | Inferensi hanya dipicu saat pengguna meminta aksi eksplisit (misal: tombol "Pecah Tugas"). Tidak ada continuous background inference. |
| **Ukuran Unduhan Model Besar (~1 GB)** | SEDANG | Unduhan hanya via koneksi Wi-Fi (opsi pengguna), mendukung *resumable download*, dan bobot model dapat dihapus oleh pengguna dari menu Pengaturan. |
| **Output JSON Model Rusak** | SEDANG | Penerapan parser JSON deterministik dengan `kotlinx.serialization`, validasi skema ketat, dan pembatasan maksimal 1x retry sebelum fallback ke template manual. |
| **Kekecewaan Fitur Kolaborasi Tim** | RENDAH | Komunikasi transparan sejak awal bahwa MVP berfokus pada produktivitas akademik pribadi dan tim lokal (*privacy-first*), dengan fitur ekspor tim yang mudah dibagikan via WhatsApp/Telegram. |

---

## 8. Rencana Implementasi Bertahap (Phased Roadmap)

1. **Fase 1 — Documentation & ADR Lock (Selesai pada langkah ini):**  
   Mengunci ADR-014 hingga ADR-019, memperbarui master indeks, menghapus folder `backend/`, dan merevisi seluruh dokumentasi arsitektur.
2. **Fase 2 — Android Data & FTS5 Local Engine:**  
   Menyiapkan entitas Room, tabel virtual SQLite FTS5, DataStore, dan Deterministic Priority Engine di proyek Android.
3. **Fase 3 — LiteRT-LM & On-Device Gemma Integration:**  
   Menyusun modul abstraksi `AiEngine`, implementasi `LiteRtLmEngine`, `ModelDownloadManager`, dan validasi integritas checksum SHA-256.
4. **Fase 4 — Local RAG Pipeline & Task Feature Wiring:**  
   Menghubungkan `LocalRagPipeline` (FTS5 retrieval) dengan use case Intelligent Task Breakdown dan Deadline Risk Analysis.
5. **Fase 5 — UI/UX Neo-Bauhaus & Validation:**  
   Membangun antarmuka Jetpack Compose, pengujian unit test deterministik, pengujian integrasi lokal, dan audit keamanan Play Store.
