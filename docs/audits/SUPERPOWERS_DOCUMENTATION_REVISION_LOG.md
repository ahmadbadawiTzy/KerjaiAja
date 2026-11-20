# Log Revisi Dokumentasi Resmi (Superpowers Documentation Revision Log) — KerjaiAja

> **Proyek:** KerjaiAja (Android Native Academic Productivity Platform)  
> **Workspace:** `/home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja`  
> **Repositori:** [`ahmadbadawiTzy/KerjaiAja`](https://github.com/ahmadbadawiTzy/KerjaiAja)  
> **Branch:** `main`  
> **Tanggal Revisi:** 2026-11-20  
> **Baseline Audit Historis:** [`docs/audits/SUPERPOWERS_DOCUMENTATION_AUDIT.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/audits/SUPERPOWERS_DOCUMENTATION_AUDIT.md)  
> **Status Audit Baseline:** *Dipertahankan penuh sebagai artefak historis yang tidak boleh ditimpa atau diubah.*  

---

## 1. Ikhtisar Revisi (Revision Overview)

Dokumen ini mencatat seluruh pembaruan spesifikasi teknis, arsitektur, dan rencana implementasi yang dilakukan berdasarkan **empat keputusan rekayasa yang telah disetujui pengguna**, serta mendokumentasikan penanganan terhadap usulan yang **belum disetujui pengguna** (*Group Project*).

---

## 2. Empat Keputusan Rekayasa yang Disetujui (Approved Decisions)

### DECISION 01 — Local AI Capability Detection
- **Dasar Keputusan:** [ADR-001: Local AI Capability Detection](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-001-LOCAL-AI-CAPABILITY-DETECTION.md)
- **Ringkasan:**
  1. Perangkat RAM 4 GB menggunakan mode fallback manual konvensional secara default demi mencegah crash pemutusan paksa oleh Android *Low Memory Killer* (LMKD).
  2. Perangkat RAM 6 GB ditetapkan sebagai **kandidat pengujian (*test candidate*)** bersyarat.
  3. Perangkat RAM >= 8 GB ditetapkan sebagai **kandidat prioritas pengujian (*primary target*)**.
  4. Kelayakan aktivasi AI dievaluasi dinamis saat runtime (*Adaptive Multi-Factor Detection*): memeriksa memori bebas riil (`availMem > 1.5 GB`), storage (> 2.5 GB), suhu (*thermal headroom*), dan status baterai (> 20%).
  5. Seluruh angka RAM fisik tidak dijadikan jaminan bebas crash tanpa validasi profil memori di perangkat fisik nyata.

### DECISION 02 — Task-Based Local RAG MVP Scope
- **Dasar Keputusan:** [ADR-002: Task-Based Local RAG MVP Scope](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-002-TASK-BASED-LOCAL-RAG-MVP.md)
- **Ringkasan:**
  1. Menghilangkan ketergantungan MVP v1.0 terhadap modul dan entitas catatan kuliah (`academic_notes` dan `academic_notes_fts`).
  2. Menunda modul catatan kuliah (`academic_notes`) secara resmi ke rilis pasca-MVP (v1.x / v2.0).
  3. Menyusun konteks Local RAG pada MVP secara eksklusif dari data tugas yang tersedia (`tasks`: judul, deskripsi instruksi, kode mata kuliah, dan riwayat tugas sejenis via `tasks_fts`).
  4. Menerapkan batas akses data (*data minimization*): model AI hanya membaca teks yang disuplai oleh UseCase tanpa akses query bebas ke database Room.

### DECISION 03 — Structured AI Output Strategy & Defense-in-Depth
- **Dasar Keputusan:** [ADR-003: Structured AI Output Strategy](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-003-STRUCTURED-AI-OUTPUT.md)
- **Ringkasan:**
  1. Memverifikasi kemampuan resmi runtime LiteRT-LM bertahap sebelum mewajibkan native grammar-constrained decoding.
  2. Menerapkan pipeline pertahanan 5 lapis (*Defense-in-Depth*): Strict System Prompt -> Sanitizer Pra-Parsing (pembersihan markdown ticks ```json, preamble, koma gantung) -> Validasi Skema Kotlinx Serialization -> Bounded Retry (1x) -> Deterministic Manual Fallback.
  3. Menghapus klaim angka validitas 98% tanpa bukti empiris; memberi label `REQUIRES TECHNICAL VALIDATION` pada seluruh metrik kualitas output AI hingga dataset uji dievaluasi di perangkat fisik.

### DECISION 04 — Adaptive AI Session Lifecycle
- **Dasar Keputusan:** [ADR-004: Adaptive AI Session Lifecycle](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-004-ADAPTIVE-AI-SESSION-LIFECYCLE.md)
- **Ringkasan:**
  1. Mengadopsi State Machine adaptif 7 status: `UNINITIALIZED`, `LOADING`, `READY`, `RUNNING_INFERENCE`, `IDLE`, `RELEASING_RESOURCES`, `ERROR`.
  2. Menggantikan aturan pelepasan memori instan dengan *Configurable Idle Timeout* (default 90–120 detik) selama pengguna berada di layar AI untuk mengeliminasi friksi cold-load 5.5 detik saat regenerasi respon.
  3. Menjamin pembersihan sumber daya aman dan proaktif saat: pengguna berpindah layar (*screen exit*), aplikasi masuk latar belakang (*onStop*), atau sistem mengirimkan sinyal tekanan memori (`ComponentCallbacks2.onTrimMemory`).
  4. Menyediakan dukungan pembatalan Coroutine Job terstruktur (*cancellation token*) saat inferensi dibatalkan pengguna.

---

## 3. Status Keputusan Tertunda / Belum Disetujui (Unapproved Decision)

### PEND-001: Usulan Penundaan Fitur Proyek Kelompok (Group Project) ke v1.2
- **Status:** **`PENDING USER DECISION`** *(Belum Disetujui Pengguna)*
- **Tindakan yang Diambil:**
  - **TIDAK memindahkan** Group Project ke rilis v1.2.
  - **TIDAK menghapus** fitur Group Project dari lingkup MVP v1.0.
  - **TIDAK mengubah** backlog (`GRP-01` s/d `GRP-03`) atau alur implementasi Tahap 4 terkait proyek kelompok.
  - Seluruh dokumen mempertahankan spesifikasi awal: pelacakan peran sub-tugas kelompok pada basis data lokal dan ekspor ringkasan progres ke format teks pesan.
  - Catatan status ditambahkan secara transparan di `MVP_SCOPE.md`, `PRODUCT_REQUIREMENTS.md`, `BACKLOG.md`, dan `IMPLEMENTATION_PHASES.md` bahwa usulan penundaan fitur ini berstatus *Pending User Decision*.

---

## 4. Daftar Berkas yang Diperbarui & Ringkasan Perubahan

| Berkas | Jenis Perubahan | Ringkasan Perubahan & Alasan Teknis |
| :--- | :---: | :--- |
| `docs/decisions/adr/ADR-001-LOCAL-AI-CAPABILITY-DETECTION.md` | **File Baru** | ADR resmi klasifikasi tiering RAM (4GB default fallback, 6GB kandidat uji, 8GB+ prioritas) dan inspeksi multi-faktor. |
| `docs/decisions/adr/ADR-002-TASK-BASED-LOCAL-RAG-MVP.md` | **File Baru** | ADR resmi pembatasan RAG MVP pada entitas `tasks` dan penundaan modul `academic_notes` ke pasca-MVP. |
| `docs/decisions/adr/ADR-003-STRUCTURED-AI-OUTPUT.md` | **File Baru** | ADR resmi verifikasi bertahap LiteRT-LM, pipeline defense-in-depth 5 lapis, dan pelabelan validasi empiris. |
| `docs/decisions/adr/ADR-004-ADAPTIVE-AI-SESSION-LIFECYCLE.md` | **File Baru** | ADR resmi State Machine 7 status, configurable idle timeout, dan integrasi `onTrimMemory`. |
| `docs/decisions/ARCHITECTURE_DECISIONS.md` | Diperbarui | Menambahkan indeks link ke 4 ADR modular baru dan mencatat PEND-001 (Group Project pending decision). |
| `docs/product/MVP_SCOPE.md` | Diperbarui | Menambahkan baris penundaan `academic_notes` ke pasca-MVP, memperjelas RAG berbasis tasks & adaptive capability detection, mencatat status Group Project, dan menyelaraskan kriteria sukses. |
| `docs/product/PRODUCT_REQUIREMENTS.md` | Diperbarui | FR-05 diselaraskan dengan RAG berbasis tasks; NFR-02 diperbarui dengan multi-factor capability detector dan adaptive session lifecycle; FR-06 dianotasi status pending decision. |
| `docs/architecture/DATA_ARCHITECTURE.md` | Diperbarui | Menandai entitas `academic_notes` sebagai pasca-MVP; mengganti FTS MVP menjadi `tasks_fts`; menegaskan batas akses data AI read-only minimal. |
| `docs/architecture/SYSTEM_ARCHITECTURE.md` | Diperbarui | Komponen FTS diarahkan ke `tasks_fts`; deskripsi Adaptive Local AI Subsystem dan manajemen memori diselaraskan dengan ADR-001 s/d ADR-004. |
| `docs/architecture/ANDROID_ARCHITECTURE.md` | Diperbarui | Menambahkan Type-Safe Navigation Compose (Navigation 2.8+ berbasis `@Serializable`); memperbarui Data Layer Local AI Engine Wrapper dengan lifecycle manager dan output sanitizer. |
| `docs/ai/LOCAL_AI_ARCHITECTURE.md` | Diperbarui Menyeluruh | Diagram Mermaid alur kerja diperbarui (multi-factor check & task RAG); ditambahkan diagram State Machine session lifecycle; memperjelas 5 komponen subsistem AI. |
| `docs/ai/MODEL_STRATEGY.md` | Diperbarui | Matriks keputusan diperbarui dengan tiering RAM realistis dan label `REQUIRES TECHNICAL VALIDATION`; metodologi prompt diselaraskan dengan defense-in-depth dan task RAG. |
| `docs/ai/MODEL_DISTRIBUTION.md` | Diperbarui | Mengganti format nama berkas model dari `.bin` ke format resmi LiteRT-LM `.litertlm`; menambahkan validasi ketersediaan storage minimal 2.5 GB sebelum unduhan. |
| `.gitignore` | Diperbarui | Menambahkan aturan pengabaian berkas `*.litertlm` dan `assets/models/*.litertlm`. |
| `docs/ai/AI_EVALUATION_STRATEGY.md` | Diperbarui Menyeluruh | Memberikan label `REQUIRES TECHNICAL VALIDATION` pada SLA numerik; memperbarui peak memory target ke 800MB-1.0GB; menjabarkan pipeline defense-in-depth dan protokol benchmark tiering. |
| `planning/BACKLOG.md` | Diperbarui | AI-02 (multi-factor detector), AI-03 (.litertlm), AI-05 (`tasks_fts`), AI-06 (lifecycle manager), AI-07 (output parser defense-in-depth), AI-09 (onTrimMemory fallback); anotasi status Group Project. |
| `planning/IMPLEMENTATION_PHASES.md` | Diperbarui | Tahap 3 diselaraskan penuh dengan aktivitas ADR-001 s/d ADR-004 dan DoD terproteksi; Tahap 4 dianotasi status pending decision Group Project. |
| `docs/development/TESTING_STRATEGY.md` | Diperbarui | Menambahkan pengujian Room Migration (`MigrationTestHelper`), WorkManager worker test, pengujian state machine lifecycle AI, multi-factor detector test, dan 50+ fixture output malformed. |
| `README.md` | Diperbarui | Bagian 6 (Strategi Local AI) diselaraskan penuh dengan deteksi kapabilitas adaptif, task-based RAG, format `.litertlm`, defense-in-depth, dan session lifecycle. |
| `PROJECT_STATUS.md` | Diperbarui | Matriks Kesiapan Fondasi diselaraskan dengan ADR-001 s/d ADR-004; Next Milestones diperbarui menuju Tahap 1 bootstrapping. |

---

## 5. Sumber Teknis yang Diverifikasi Resmi

1. **Google AI Edge LiteRT-LM:**
   - Dokumentasi: [https://ai.google.dev/edge/litert](https://ai.google.dev/edge/litert)
   - SDK Artifact: `com.google.ai.edge.litertlm:litertlm-android`
   - Format Model: Berkas paket `.litertlm` (terverifikasi via HuggingFace Google LiteRT Community & Android AI Edge showcase).
2. **Google DeepMind Gemma 3 (Maret 2025):**
   - Varian: 1B, 4B, 12B, 27B.
   - Varian 1B: Teks-only, context window 32K token, 2T token training multilingual, arsitektur dense decoder-only.
3. **Android Developers Documentation:**
   - App Architecture Guide (UDF, Clean Architecture, Repository Pattern).
   - Type-Safe Navigation in Compose (AndroidX Navigation 2.8.0+ berbasis Kotlin Serialization).
   - Android Process Lifecycle & LMKD: [https://developer.android.com/guide/components/activities/process-lifecycle](https://developer.android.com/guide/components/activities/process-lifecycle)
   - Memory Management & `ComponentCallbacks2.onTrimMemory()`: [https://developer.android.com/topic/performance/memory](https://developer.android.com/topic/performance/memory)
4. **OWASP Mobile Application Security:**
   - Mobile ASVS v5.0 (MASVS-STORAGE & MASVS-CODE).

---

## 6. Asumsi Teknis yang Belum Diverifikasi Fisik (`REQUIRES TECHNICAL VALIDATION`)

Daftar parameter yang secara eksplisit ditandai belum memiliki bukti pengukuran empiris dan wajib divalidasi pada perangkat fisik di Tahap 3:

1. **Generation Throughput (TPS):** Target `>= 8-12 tokens/detik` pada chipset kelas menengah (Snapdragon 680 / 7s Gen 2 / Dimensity 7050).
2. **Time to First Token (TTFT):** Target `<= 2.5 detik` saat kondisi model sudah berada di status `READY`/`IDLE` (Warm Inference).
3. **Peak Memory Footprint:** Lonjakan alokasi memori aktual saat context window 1.024–2.048 token aktif di RAM native (apakah berada di bawah 1.0 GB).
4. **Thermal Throttling:** Kenaikan suhu baterai selama 5 kali pemanggilan beruntun di iklim tropis Indonesia (apakah tetap `<= 2.5°C`).
5. **Akurasi JSON Output Model 1B:** Tingkat keberhasilan pembersihan format teks model menggunakan sanitizer regex sebelum mencapai bounded retry.

---

## 7. Temuan Audit yang Belum Diselesaikan / Ditunda

1. **AUD-HIGH-01 (Group Project vs Offline Scope):**
   - *Status:* **DITAHAN / PENDING USER DECISION**. Pengguna belum menyetujui penundaan modul ini ke v1.2. Keputusan tetap mempertahankan spesifikasi awal hingga ada instruksi baru.
2. **AUD-MED-04 (Database Encryption at-rest via SQLCipher):**
   - *Status:* **DITUNDA**. Untuk MVP v1.0, Room menggunakan SQLite standar tanpa SQLCipher demi menghemat konsumsi CPU/baterai, bergantung pada proteksi sandbox bawaan OS Android.

---

## 8. Matriks Konsistensi Antar Dokumen Pasca-Revisi (Consistency Matrix)

| Area Evaluasi | Dokumen Terkait | Status Konsistensi | Penjelasan Rekonsiliasi |
| :--- | :--- | :---: | :--- |
| **Ruang Lingkup MVP vs Requirements** | `MVP_SCOPE.md` vs `PRODUCT_REQUIREMENTS.md` | ✅ **KONSISTEN** | Keduanya menegaskan RAG MVP berbasis data tasks; `academic_notes` resmi ditunda ke pasca-MVP; Group Project ditandai pending decision. |
| **Arsitektur Data vs RAG AI** | `DATA_ARCHITECTURE.md` vs `LOCAL_AI_ARCHITECTURE.md` | ✅ **KONSISTEN** | Keduanya menggunakan `tasks_fts` sebagai virtual table FTS MVP; `academic_notes` ditandai pasca-MVP pada kedua dokumen. |
| **AI Capability vs Perangkat Keras** | `MODEL_STRATEGY.md` vs `AI_EVALUATION_STRATEGY.md` | ✅ **KONSISTEN** | Keduanya menyelaraskan tiering RAM (4GB default fallback, 6GB kandidat uji, 8GB+ prioritas uji) dan memberi label `REQUIRES TECHNICAL VALIDATION`. |
| **Keluaran Terstruktur AI** | `LOCAL_AI_ARCHITECTURE.md` vs `TESTING_STRATEGY.md` | ✅ **KONSISTEN** | Keduanya mengadopsi pipeline 5 lapis defense-in-depth, pengujian 50+ fixture malformed, dan transisi ke manual fallback tanpa crash. |
| **Siklus Hidup Model AI** | `LOCAL_AI_ARCHITECTURE.md` vs `ANDROID_ARCHITECTURE.md` | ✅ **KONSISTEN** | Keduanya menerapkan State Machine 7 status adaptif, configurable idle timeout, dan integrasi callback `onTrimMemory`. |
| **Backlog & Rencana Fase** | `BACKLOG.md` vs `IMPLEMENTATION_PHASES.md` | ✅ **KONSISTEN** | Item AI-01 s/d AI-09 dan Tahap 3 sepenuhnya sinkron dengan 4 ADR baru tanpa memuat entitas catatan yang belum ada. |
| **Format Ekstensi Berkas Model** | `MODEL_DISTRIBUTION.md` vs `.gitignore` | ✅ **KONSISTEN** | Keduanya telah menyepakati format resmi `.litertlm` dan mengecualikannya dari pelacakan git. |

---
*Log revisi ini disusun dengan mematuhi prinsip ketat audit trail rekayasa Superpowers.*
