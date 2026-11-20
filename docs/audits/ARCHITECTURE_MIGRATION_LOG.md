# Log Migrasi Arsitektur (Architecture Migration Log) — KerjaiAja

Dokumen ini mencatat keputusan dan justifikasi teknis di balik seluruh tahapan migrasi arsitektur pada proyek **KerjaiAja**.

---

## 1. Migrasi Pertama: Full Product & Tech Stack Rebaseline (2026-11-20, Sesi Pagi)
- **Konteks:** Transisi awal dari eksperimen on-device tanpa kolaborasi menuju hybrid monolith (Go + Gin + Supabase + self-hosted Qwen3 via `llama.cpp`).
- **Pelajaran Rekayasa:** Memetakan pentingnya 3 pilar produk (Personal, Prioritization, Team Collaboration) dan bahaya ketergantungan pada API komersial berbayar.

---

## 2. Fundamental Architecture Rebaseline: On-Device Google Gemma & Pure Local (2026-11-20, Sesi Sore)
- **Konteks:** Penghapusan backend Go/Gin dan server VPS, beralih ke pure on-device Google Gemma via LiteRT-LM dan Local RAG SQLite FTS5 (ADR-014 s/d ADR-019).
- **Pelajaran Rekayasa:** Berhasil membebaskan aplikasi dari beban biaya server dan ketergantungan API pihak ketiga, namun memunculkan keterbatasan pada sinkronisasi lintas perangkat dan kolaborasi tim real-time.

---

## 3. Hybrid Architecture Rebaseline: Supabase BaaS + Optional On-Device Gemma AI (2026-11-20, Sesi Malam)

Pengguna secara sadar menetapkan arah arsitektur final: **Hybrid Offline-First Productivity Platform (Supabase Managed BaaS + Optional On-Device Google Gemma AI)**.

### 3.1 Matriks Keputusan Migrasi Definitif

| Aspek Arsitektur | Pure Local (Sore) | Hybrid Offline-First (Definitif) | Justifikasi Rekayasa |
| :--- | :--- | :--- | :--- |
| **Model Pengalaman** | Single Offline Experience | **Dual Experience (Mode A & Mode B)** | Mode A (100% tanpa AI) dan Mode B (Asisten Kerja Opsional) inklusif untuk semua perangkat. |
| **Penyimpanan Lokal** | Room Database + FTS5 | **Room Database + FTS5 (Local-Write-First)** | Respons instan milidetik, UI tidak pernah terblokir latency jaringan. |
| **Penyimpanan Cloud** | Tidak ada | **Supabase PostgreSQL via SDK Resmi** | Menyimpan backup dan sinkronisasi multi-device via `supabase-kt` v3.9.0. |
| **Custom Backend** | Dihapus | **TIDAK ADA BACKEND CUSTOM (Zero Backend)** | Tidak ada Go/Gin/Node/Python server; Supabase digunakan sebagai BaaS langsung dari Android. |
| **Autentikasi** | Lokal murni | **Guest-First & Supabase Auth Opsional** | Mahasiswa bisa langsung memakai tanpa login; migrasi data transparan saat mendaftar. |
| **Sinkronisasi Data** | Tidak ada | **Offline-First Sync via WorkManager** | Antrean `sync_pending_queue`, Last-Write-Wins (LWW), dan soft deletion tombstone (`deleted_at`). |
| **Kolaborasi Tim** | Berbagi file `.kerjai` | **Supabase Realtime & PostgreSQL RLS** | Sinkronisasi status tugas kelompok instan via WebSocket CDC dan proteksi RLS multi-tenant. |
| **Inferensi AI** | On-Device Gemma | **On-Device Gemma 3 1B IT & 270M (LiteRT-LM)** | 100% on-device, privasi terjamin, zero external AI API, dilarang generic chatbot. |
| **Kelayakan Hardware** | Pengecekan manual | **DeviceCapabilityManager Multi-Faktor** | Evaluasi RAM bebas ($\ge 1.5$ GB), 64-bit ABI, storage, thermal, dan baterai. |
| **Penyimpanan Bobot AI** | Internal disk | **Internal Disk Sandbox (No Supabase Storage)** | Bobot AI dilarang diunggah ke Supabase Storage; diunduh dari Google AI Edge CDN. |
| **Biaya Operasional** | Rp0 | **Rp 0 / $0 (Supabase Free Tier Guardrails)** | Pengendalian kuota 500 MB DB, 1 GB Storage, 2 GB egress bulanan. |

---

## 4. Resolusi ADR yang Terkait

1. **ADR-020 (Supabase Managed BaaS Integration):** Menggantikan ADR-003 dan ADR-017; menetapkan integrasi Supabase resmi langsung dari Android.
2. **ADR-021 (Guest-First & Optional Cloud Authentication):** Mengizinkan onboarding seketika tanpa login dengan migrasi data transparan.
3. **ADR-022 (Offline-First Sync & Conflict Resolution):** Menetapkan pola local-write-first di Room, antrean `sync_pending_queue`, dan resolusi konflik LWW.
4. **ADR-023 (Dual Experience Product Architecture):** Memisahkan pengalaman Mode A (tanpa AI) dan Mode B (dengan Asisten Kerja opsional).
5. **ADR-024 (Device Capability Management):** Evaluasi hardware multi-faktor sebelum mengizinkan unduhan model on-device.
6. **ADR-025 (Asisten Kerja Boundaries & Chatbot Ban):** Membatasi AI pada 5 use-case tugas kontekstual dan melarang generic chatbot.
7. **ADR-026 (Team Collaboration via Supabase Realtime & RLS):** Menggantikan ADR-006 dan ADR-018; menetapkan kolaborasi tim cloud via Supabase Realtime WebSocket dan PostgreSQL RLS.

---

## 5. Dokumen Arsitektur & Perencanaan yang Telah Diselaraskan

1. **Dokumen Audit Baru:** [HYBRID_ARCHITECTURE_REBASELINE_AUDIT.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/audits/HYBRID_ARCHITECTURE_REBASELINE_AUDIT.md).
2. **Dokumen Arsitektur Baru:**
   - [SUPABASE_ARCHITECTURE.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/SUPABASE_ARCHITECTURE.md)
   - [SUPABASE_FREE_TIER_AND_COST_GUARDRAILS.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/SUPABASE_FREE_TIER_AND_COST_GUARDRAILS.md)
   - [SYNC_ARCHITECTURE.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/SYNC_ARCHITECTURE.md)
   - [DEVICE_CAPABILITY_STRATEGY.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/ai/DEVICE_CAPABILITY_STRATEGY.md)
   - [AI_ASSISTANT_SPECIFICATION.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/ai/AI_ASSISTANT_SPECIFICATION.md)
   - [SECURITY_ARCHITECTURE.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/security/SECURITY_ARCHITECTURE.md)
3. **Dokumen Master ADR:** [ARCHITECTURE_DECISIONS.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/ARCHITECTURE_DECISIONS.md).
4. **Dokumen Produk:** `PRODUCT_VISION.md`, `PRODUCT_REQUIREMENTS.md`, `MVP_SCOPE.md`.
5. **Dokumen Arsitektur:** `SYSTEM_ARCHITECTURE.md`, `ANDROID_ARCHITECTURE.md`, `DATA_ARCHITECTURE.md`, `DEPLOYMENT_ARCHITECTURE.md`, `API_DESIGN.md`.
6. **Dokumen AI:** `AI_ARCHITECTURE.md`, `MODEL_STRATEGY.md`, `LOCAL_RAG_ARCHITECTURE.md`, `ON_DEVICE_MODEL_EVALUATION.md`, `AI_EVALUATION_STRATEGY.md`.
7. **Dokumen Kinerja & Keamanan:** `APP_SIZE_AND_RESOURCE_BUDGET.md`, `PLAY_STORE_READINESS.md`.
8. **Dokumen Perencanaan & Status:** `DEVELOPMENT_ROADMAP.md`, `TESTING_STRATEGY.md`, `BACKLOG.md`, `IMPLEMENTATION_PHASES.md`, `README.md`, `PROJECT_STATUS.md`.

---

## 4. Final Rebaseline: Hybrid Offline-First + AI Learning Assistant + Photo Question Solver (2026-11-20, Sesi Akhir)

- **Konteks:** Perluasan kapabilitas Mode B untuk menyertakan fitur edukasi akademik unggulan **Foto Soal & Pembahasan** dengan pemrosesan efemeral lokal.
- **Pembaruan Keputusan Rekayasa (ADR-027 s/d ADR-032):**
  1. **ADR-027:** Spesifikasi Fitur Foto Soal & Pembahasan (7 bagian terstruktur, 12 rumpun mata pelajaran, kamera/galeri, crop/rotate/retake).
  2. **ADR-028:** Pemrosesan OCR-First On-Device (Google ML Kit Text Recognition v2 + Human-in-the-Loop Review Screen + Gemma Text Model).
  3. **ADR-029:** Pemisahan Kapabilitas Hardware: `TextCapability` (RAM $\ge 4$ GB) vs `VisionCapability` (RAM $\ge 8$ GB, NPU $\ge 15$ TOPS).
  4. **ADR-030:** Strategi Evaluasi Model Vision Multimodal (**Gemma 3n E2B**) sebagai Jalur B Laboratorium Khusus Perangkat Tinggi (Opt-in terpisah).
  5. **ADR-031:** Kendali Mutu & Validasi Solusi Akademik Deterministik (`AnswerValidator`: verifikasi 7 bagian, kalkulasi aritmetika, konsistensi satuan, uncertainty flagging).
  6. **ADR-032:** Perlindungan Privasi Citra & Siklus Hidup Efemeral (Penyimpanan citra di `cacheDir`, zero cloud image upload, penghapusan instan, UU PDP & Play Store Families compliance).
- **Dokumen Baru yang Dihasilkan:**
  - `docs/ai/IMAGE_PROCESSING_ARCHITECTURE.md`
  - `docs/ai/OCR_STRATEGY.md`
  - `docs/ai/MULTIMODAL_MODEL_EVALUATION.md`
  - `docs/ai/AI_OUTPUT_VALIDATION.md`
  - `docs/quality/SUBJECT_BENCHMARK.md`
  - `docs/quality/OCR_EVALUATION.md`
  - `docs/quality/DEVICE_BENCHMARK.md`
  - `docs/security/PRIVACY_REVIEW.md`
  - `docs/security/SECURITY_REVIEW.md`

---

## 5. Progressive AI Processing Architecture & Lifecycle Lock (2026-11-20, Sesi Lanjutan)

- **Konteks:** Menjawab kebutuhan efisiensi energi, stabilitas memori, dan eliminasi latensi tidak perlu melalui doktrin: *"Lakukan sebanyak mungkin dengan pemrosesan deterministik ringan sebelum memanggil model AI."*
- **Pembaruan Keputusan Rekayasa (ADR-033 s/d ADR-035):**
  1. **ADR-033 (Progressive AI Processing Pipeline):** Mengunci alur pemrosesan 7 tahap (Stage 0: Deterministic -> Stage 1: Input Preprocessing -> Stage 2: Task Router -> Stage 3: OCR-First -> Stage 4: Local RAG -> Stage 5: LiteRT-LM Inference -> Stage 6: Subject Validation).
  2. **ADR-034 (On-Device Model Lifecycle & Memory Management):** Menetapkan arsitektur lazy loading, proteksi single active session `Mutex`, idle auto-unload 5 menit, batas inferensi 15 detik, dan callback `onTrimMemory(15)`.
  3. **ADR-035 (Per-Model Device Capability Evaluation):** Transisi dari status kelayakan global menjadi evaluasi spesifik per-model dan per-kemampuan (`ML Kit OCR`, `Gemma 270M`, `Gemma 1B IT`, `Gemma 3n E2B`).
- **Dokumen Baru yang Dihasilkan:**
  - `docs/architecture/PROGRESSIVE_AI_PROCESSING.md`
  - `docs/architecture/AI_MODEL_LIFECYCLE.md`
  - `docs/architecture/DEVICE_CAPABILITY_MANAGER.md`
  - `docs/performance/AI_PERFORMANCE_BENCHMARK.md`
  - `docs/testing/PHOTO_QUESTION_SOLVER_TEST_PLAN.md`
  - `docs/decisions/adr/ADR-033-PROGRESSIVE-AI-PROCESSING-PIPELINE.md`
  - `docs/decisions/adr/ADR-034-ON-DEVICE-MODEL-LIFECYCLE-AND-MEMORY-MANAGEMENT.md`
  - `docs/decisions/adr/ADR-035-PER-MODEL-DEVICE-CAPABILITY-EVALUATION.md`
- **Scaffold & Unit Test yang Tervalidasi:**
  - `app/src/main/java/com/kerjaiaja/app/core/router/TaskClassificationRouter.kt`
  - `app/src/main/java/com/kerjaiaja/app/core/lifecycle/ModelLifecycleManager.kt`
  - `app/src/main/java/com/kerjaiaja/app/core/solver/SubjectAdaptiveValidator.kt`
  - `app/src/main/java/com/kerjaiaja/app/core/device/DeviceCapabilityManager.kt`
  - 41 unit tests lulus 100% di `app/src/test/`.

---

## 6. Complete Ecosystem Rebaseline: Progressive AI + Web Admin CMS (2026-11-20, Sesi Final)

- **Konteks:** Menjawab kebutuhan tata kelola operasional kurikulum, penanganan keluhan feedback/bug report pengguna, pemantauan kesehatan kuota Free Tier Supabase, dan audit trail administratif melalui antarmuka web khusus tanpa menambah server backend kustom.
- **Pembaruan Keputusan Rekayasa (ADR-036 s/d ADR-038):**
  1. **ADR-036 (Web Admin CMS Architecture & Technology Stack):** Menetapkan arsitektur dashboard web berbasis Next.js 15 App Router, React 19, TypeScript strict mode, Tailwind CSS, shadcn/ui, dan `@supabase/ssr` (Server Components by default).
  2. **ADR-037 (CMS Role-Based Access Control & Server-Side Security Model):** Menetapkan model otorisasi 3 tingkat (`SUPER_ADMIN`, `ADMIN`, `SUPPORT`), penegakan RLS di level PostgreSQL via `is_admin()`, larangan registrasi mandiri publik (invitation-only), isolasi service role key, dan audit trail otomatis pada `admin_audit_logs`.
  3. **ADR-038 (Shared Database Schema & Multi-Tenant RLS for CMS):** Menyatukan Android dan Web CMS pada satu proyek database Supabase PostgreSQL, menjaga alur Delta Sync Android (`academic_references`), isolasi data lokal Room, dan penegakan batas kuota Free Tier ($0).
- **Dokumen Baru yang Dihasilkan:**
  - `docs/product/CMS_PRODUCT_REQUIREMENTS.md`
  - `docs/architecture/WEB_ADMIN_ARCHITECTURE.md`
  - `docs/architecture/CMS_SECURITY_AND_RBAC.md`
  - `docs/architecture/CMS_DATA_ARCHITECTURE.md`
  - `docs/deployment/VERCEL_SUPABASE_DEPLOYMENT.md`
  - `docs/testing/CMS_TEST_STRATEGY.md`
  - `docs/decisions/adr/ADR-036-WEB-ADMIN-CMS-ARCHITECTURE.md`
  - `docs/decisions/adr/ADR-037-CMS-RBAC-AND-SERVER-SIDE-SECURITY.md`
  - `docs/decisions/adr/ADR-038-SHARED-DATABASE-SCHEMA-AND-CMS-RLS.md`
- **Total Architectural Decision Records:** **38 ADR Lengkap & Terkunci (ADR-001 s/d ADR-038).**

---

## 7. Knowledge Intelligence Layer & Two-Tier RAG Architecture (2026-11-20, Sesi Lanjutan)

- **Konteks:** Menjawab kebutuhan arsitektur pengambilan dan pengelolaan pengetahuan (*information retrieval & RAG*) yang profesional, aman, dapat ditelusuri (*grounded/traceable*), dan hemat sumber daya perangkat mobile.
- **Pembaruan Keputusan Rekayasa (ADR-039 s/d ADR-041):**
  1. **ADR-039 (Two-Tier Knowledge Intelligence Layer Architecture):** Memisahkan secara fisik dan logis domain *Productivity Knowledge* (`tasks_fts`) dari *Academic Knowledge* (`academic_fts`) di Room Database Android; mengunci SQLite FTS5 BM25 sebagai baseline ringan (<15 MB RAM, <20 ms latensi); menunda embedding vektor sampai pengujian nilai tambah terpenuhi.
  2. **ADR-040 (Document Ingestion, Intelligent Chunking & Source Traceability):** Menolak *fixed-size chunking*; mengadopsi segmentasi cerdas berorientasi struktur yang mempertahankan blok rumus matematika LaTeX, heading hierarki, dan header tabel; mengunci pelacakan hash SHA-256 dan sitasi terverifikasi (nama buku, bab, nomor halaman) tanpa sitasi halusinasi.
  3. **ADR-041 (Context Engineering, Prompt Injection Defense & Knowledge Sync):** Menerapkan pembatasan anggaran token ketat (Productivity $\le 800$ tokens, Academic $\le 1200$ tokens); memperlakukan dokumen hasil retrieval sebagai data pasif di `<retrieved_references>` dengan sanitasi delimiter XML; serta mengunci sinkronisasi delta luring referensi kurikulum dari CMS ke Android dengan kontrol penyimpanan mahasiswa (<100 MB).
- **Dokumen Baru yang Dihasilkan:**
  - `docs/architecture/KNOWLEDGE_INTELLIGENCE_LAYER.md`
  - `docs/architecture/LOCAL_RETRIEVAL_ENGINE.md`
  - `docs/architecture/DOCUMENT_INGESTION_PIPELINE.md`
  - `docs/architecture/CONTEXT_ENGINEERING.md`
  - `docs/architecture/KNOWLEDGE_SYNC.md`
  - `docs/architecture/RAG_SECURITY_AND_PRIVACY.md`
  - `docs/product/ACADEMIC_KNOWLEDGE_REQUIREMENTS.md`
  - `docs/product/CMS_KNOWLEDGE_MANAGEMENT.md`
  - `docs/testing/RAG_EVALUATION_PLAN.md`
  - `docs/performance/RAG_BENCHMARK.md`
  - `docs/decisions/adr/ADR-039-TWO-TIER-KNOWLEDGE-INTELLIGENCE-LAYER.md`
  - `docs/decisions/adr/ADR-040-DOCUMENT-INGESTION-INTELLIGENT-CHUNKING-TRACEABILITY.md`
  - `docs/decisions/adr/ADR-041-CONTEXT-ENGINEERING-PROMPT-INJECTION-AND-KNOWLEDGE-SYNC.md`
- **Total Architectural Decision Records:** **41 ADR Lengkap & Terkunci (ADR-001 s/d ADR-041).**



