# Ruang Lingkup MVP (Minimum Viable Product Scope) — KerjaiAja

Dokumen ini mendefinisikan batasan ruang lingkup rilis perdana (**KerjaiAja MVP v1.0**) pasca-rebaseline arsitektur hybrid (**Hybrid Offline-First: Supabase Cloud Services + Optional On-Device Google Gemma AI**).

---

## 1. Prinsip Penentuan Ruang Lingkup (Scoping Principles)

1. **Dual Experience Architecture:**
   - **Mode A (Produktivitas Penuh Tanpa AI):** 100% fungsionalitas manajemen tugas, kalender, mata kuliah, deterministik skor prioritas (0–100), sinkronisasi cloud Supabase (jika login), dan kolaborasi tim berfungsi tanpa membutuhkan model AI.
   - **Mode B (Produktivitas dengan Asisten Kerja On-Device Opsional):** Pengguna perangkat yang memenuhi syarat dapat mengaktifkan Google Gemma (1B / 270M) via LiteRT-LM untuk analisis konteks tugas lokal.
2. **Guest-First & Offline-First:** Pengguna dapat langsung menggunakan aplikasi seketika tanpa registrasi/login paksa. Seluruh penulisan data dilakukan ke Room Database terlebih dahulu (*Local-Write-First*). Sesi cloud Supabase bersifat opsional untuk sinkronisasi multi-device dan tim.
3. **Biaya Operasional Rp0 (Supabase Free Tier Guardrails):** Beroperasi penuh dalam kuota aman Supabase Free Plan (Database $\le 500\text{ MB}$, Storage $\le 1\text{ GB}$, Egress $\le 2\text{ GB/bulan}$).
4. **Bukan Chatbot Generik (Task-Contextual Assistant):** AI hadir hanya sebagai asisten terarah untuk 5 aksi kerja kontekstual. Tidak ada layar percakapan santai.
5. **Decoupled Binary & Privacy by Design:** Model AI tidak dibundel di APK, APK tetap ramping (~15–25 MB). Data prompt AI 100% diproses di perangkat lokal tanpa pernah dikirim ke cloud.

---

## 2. Matriks Ruang Lingkup Fitur (Feature Scope Matrix)

| Modul / Kapabilitas | Status MVP (v1.0) | Justifikasi Arsitektur |
| :--- | :---: | :--- |
| **Personal Task CRUD & Tagging** | ✅ **In-Scope** | Kebutuhan primer manajemen tugas akademik harian di Room DB. |
| **Subject & Schedule Management** | ✅ **In-Scope** | Pengelompokan tugas per mata kuliah dan jadwal mingguan lokal. |
| **Hierarchical Assignment Breakdown** | ✅ **In-Scope** | Dekonstruksi tugas besar menjadi sub-tugas terukur secara lokal. |
| **Study Planning & Focus Timer** | ✅ **In-Scope** | Timer fokus Pomodoro adaptif terikat pada sub-tugas aktif. |
| **Pre-Flight Submission Checklist** | ✅ **In-Scope** | Verifikasi format berkas dan kelengkapan tugas sebelum dikumpulkan. |
| **Deterministic Priority Engine** | ✅ **In-Scope** | Formula skor prioritas transparan (0–100) berbasis waktu & bobot nilai. |
| **Optional On-Device Gemma Assistant** | ✅ **In-Scope** | Penalaran kontekstual dan dekomposisi tugas via Google Gemma & LiteRT-LM. |
| **Local RAG via SQLite FTS5** | ✅ **In-Scope** | Injeksi konteks tugas relevan menggunakan BM25 langsung di ponsel. |
| **Device Capability Management** | ✅ **In-Scope** | Evaluasi kelayakan hardware multi-faktor (RAM, ABI, Storage, Thermal, Baterai). |
| **User Priority Override & Anti-Guilt UX**| ✅ **In-Scope** | Kendali penuh pengguna untuk menimpa prioritas dan aksi adaptif saat terlambat. |
| **Guest-First Onboarding & Migration**| ✅ **In-Scope** | Penggunaan instan tanpa login; migrasi data transparan saat daftar Supabase. |
| **Supabase Cloud Sync (WorkManager)** | ✅ **In-Scope** | Sinkronisasi background multi-perangkat dengan tabel `sync_pending_queue`. |
| **Team Collaboration via Supabase RLS**| ✅ **In-Scope** | Proyek kelompok akademik dengan proteksi akses multi-tenant di level database. |
| **Realtime Team Sync (WebSockets)** | ✅ **In-Scope** | Pembaruan instan tugas kelompok via Supabase Realtime (Postgres CDC). |
| **Document Attachment Storage** | ✅ **In-Scope** | Upload lampiran tugas dokumen maks 5 MB ke Supabase Storage (Private bucket). |
| **Foto Soal & Pembahasan (Jalur A)** | ⏸️ **PASCA-MVP / DITAHAN** | Ditahan menunggu keputusan pengguna. Tidak dikembangkan lebih jauh dalam MVP dasar. |
| **Vision Multimodal Lab (Jalur B)** | ⏸️ **PASCA-MVP / DITAHAN** | Ditahan menunggu keputusan pengguna. Tidak dikembangkan lebih jauh dalam MVP dasar. |
| **12 Rumpun Mata Pelajaran** | ✅ **In-Scope** | Klasifikasi kurikulum (Matematika, Fisika, Kimia, Biologi, Bhs Indonesia, Inggris, Sejarah, Geografi, Ekonomi, Sosiologi, PPKn, Lainnya). |
| **Answer Validation Layer** | ⏸️ **PASCA-MVP / DITAHAN** | Terikat dengan modul Foto Soal; ditahan menunggu kelanjutan modul Foto Soal. |
| **Ephemeral Image Privacy** | ⏸️ **PASCA-MVP / DITAHAN** | Terikat dengan modul Foto Soal; ditahan menunggu kelanjutan modul Foto Soal. |
| **Web Admin CMS (Next.js 15)** | ⏸️ **PASCA-MVP / DITAHAN** | Ditahan menunggu keputusan pengguna. Scaffold siap namun pengembangan lanjutan ditahan. |
| **Admin Audit Trail Logging** | ✅ **In-Scope** | Pencatatan otomatis setiap aksi administratif pada `admin_audit_logs`. |
| **AI Chat / Open Chatbot Screen** | ❌ **DILARANG** | KerjaiAja bukan chatbot; AI hanya untuk aksi produktivitas kontekstual. |
| **Remote AI Inference di Web CMS** | ❌ **DILARANG** | CMS tidak menjalankan inferensi AI dan tidak menyimpan external AI API keys. |
| **User Surveillance / Data Intip** | ❌ **DILARANG** | Admin CMS dilarang membaca catatan tugas atau foto dokumen privat mahasiswa. |
| **Custom Backend (Go/Gin/Node/Python)**| ❌ **DIHAPUS** | Menggunakan Supabase Managed BaaS langsung dari Android SDK dan Next.js SSR. |
| **Cloud AI Inference (Gemini/OpenAI/etc.)**| ❌ **DILARANG** | Zero external AI API. Seluruh komputasi AI berjalan on-device. |
| **AI Model Weights in Supabase Storage**| ❌ **DILARANG** | Model bobot AI wajib diunduh dari Google AI Edge / Hugging Face CDN. |

---

## 3. Kriteria Keberhasilan MVP (MVP Success Criteria)

1. **Kecepatan Pencatatan Tugas (Instant Response):** Pelajar dapat mencatat tugas baru secara lokal dalam waktu < 5 detik tanpa loading jaringan.
2. **Ketersediaan Offline Mutlak:** 100% fungsionalitas pencatatan, kalender, dan deterministik prioritas berfungsi normal saat ponsel dalam Mode Pesawat.
3. **Ukuran Unduhan APK Awal:** Ukuran APK instalasi awal berada di kisaran **$\le 25$ MB** (model AI diunduh terpisah secara on-demand).
4. **Stabilitas On-Device AI:**  
   - Zero OOM crash dengan pembatasan alokasi memori inferensi $\le 1.8\text{ GB}$.
   - Degradasi mulus (*graceful fallback*) ke template deterministik jika model AI belum diunduh.
5. **Kepatuhan Biaya Rp0:** Seluruh operasional cloud berada di bawah batas kuota Supabase Free Tier tanpa biaya hosting bulanan.
