# KerjaiAja — Academic Productivity Platform
## Hybrid Offline-First Platform · Supabase Managed BaaS · On-Device AI · Photo Question Solver · Web Admin CMS

> **Proposisi Nilai Utama:** *"Turn academic assignments into clear, manageable, and collaborative actions without guilt."*  
> Mengubah tugas akademik yang menumpuk dan membingungkan menjadi tindakan yang terstruktur, diprioritaskan secara cerdas, diselesaikan dengan bantuan pembahasan foto soal terverifikasi, dan dapat dikerjakan secara kolaboratif tanpa rasa cemas, dengan kebebasan offline-first, sinkronisasi cloud terkelola, serta tata kelola kurikulum akademik terpusat via Web Admin CMS.

---

## 1. Ikhtisar Proyek (Project Overview)

**KerjaiAja** adalah ekosistem produktivitas akademik modern yang terdiri dari dua komponen aplikasi terpadu:
1. **KerjaiAja Android Client:** Dibangun murni sebagai **Aplikasi Android Native (Kotlin + Jetpack Compose)** dengan arsitektur **Hybrid Offline-First**:
   - **Dual Experience Architecture:**
     - **Mode A (Produktivitas Penuh Tanpa AI):** 100% fungsionalitas manajemen tugas, jadwal kuliah, kalender, deterministik prioritas (0–100), sinkronisasi cloud Supabase (jika login), dan kolaborasi tim berfungsi tanpa memerlukan model AI.
     - **Mode B (Asisten Belajar & Foto Soal On-Device Opsional):** Model open-weight resmi **Google Gemma (Gemma 3 1B IT / 270M)** dijalankan langsung di perangkat via **Google AI Edge LiteRT-LM** untuk analisis tugas kontekstual dan fitur unggulan **Foto Soal & Pembahasan**.
   - **Foto Soal & Pembahasan (Dual-Track Architecture):**
     - **Jalur A (OCR-First - Universal):** Ekstraksi teks cepat via Google ML Kit Text Recognition v2 $\rightarrow$ Layar Peninjauan Pengguna (*Human-in-the-Loop Review*) $\rightarrow$ Model Teks Gemma 3 (1B IT / 270M) $\rightarrow$ *Answer Validation Layer* deterministik menghasilkan solusi 7 bagian terstruktur untuk 12 rumpun mata pelajaran.
     - **Jalur B (Laboratorium Multimodal - Khusus Flagship):** Evaluasi eksperimental model vision **Gemma 3n E2B** untuk soal diagram geometri dan grafik sains pada perangkat dengan RAM $\ge 8$ GB dan NPU $\ge 15$ TOPS.
     - **Privasi Citra Efemeral:** Berkas citra foto diproses 100% lokal di `cacheDir` dan langsung dihapus seketika setelah inferensi (zero cloud upload foto soal).
   - **Offline-First & Local-Write-First:**
     - Seluruh operasi CRUD ditulis langsung ke **Room Database 2.6.1** lokal terlebih dahulu (respons seketika milidetik).
     - Sinkronisasi cloud berjalan di background melalui WorkManager `SyncWorker` dengan antrean `sync_pending_queue`.
     - Onboarding **Guest-First**: Mahasiswa dapat langsung memakai aplikasi tanpa registrasi; migrasi data transparan saat mendaftar akun Supabase.
   - **Zero AI Cloud Compute & Zero External AI API:**
     - Dilarang keras memanggil Gemini API, OpenAI API, Anthropic, atau Groq. Data prompt AI 100% diproses di ponsel.
2. **KerjaiAja Web Admin CMS:** Dashboard administrasi modern berbasis web untuk mengelola kurikulum akademik, menangani masukan pengguna, dan memantau kesehatan operasional platform:
   - **Modern Web Stack:** Next.js 15 (App Router), React 19, TypeScript (*strict mode*), Tailwind CSS, shadcn/ui, dan Radix UI.
   - **Server-Side Security & RBAC:** Autentikasi aman berbasis HTTP-only cookies via `@supabase/ssr`, peran admin terisolasi (`SUPER_ADMIN`, `ADMIN`, `SUPPORT`) di tabel `public.admin_roles` (bukan di metadata klien), dan audit log otomatis (`admin_audit_logs`).
   - **Strict Anti-Surveillance & Privacy:** RLS PostgreSQL secara tegas melarang admin CMS melihat atau membaca isi tugas pribadi, subtask, lampiran catatan, atau riwayat foto soal mahasiswa. CMS murni untuk tata kelola operasional.
   - **Zero Custom Backend:** Langsung berkomunikasi dengan Supabase BaaS via PostgREST yang diproteksi RLS.
3. **Shared Backend:** **Supabase Managed BaaS** (Auth, PostgREST, Realtime, Storage) beroperasi pada batas aman kuota Free Tier ($0).

---

## 2. Lima Pilar Inti Produk (The Five Core Pillars)

1. **Personal Academic Productivity (Pilar 1):**  
   Pengelolaan tugas mandiri dan mata kuliah dengan basis data lokal Room Database (*offline-first*), dekonstruksi tugas berjenjang (*hierarchical assignment breakdown*), pengatur waktu fokus (*Pomodoro-friendly timer*), dan verifikasi berkas sebelum dikumpulkan (*pre-flight submission checklist*).
2. **Photo Question Solver & Learning Assistant (Pilar 2):**  
   Solusi pembahasan akademik terstruktur 7 bagian via foto dokumen tugas/soal untuk 12 rumpun mata pelajaran, didukung OCR on-device dan verifikasi jawaban deterministik.
3. **Intelligent & Explainable Task Prioritization (Pilar 3):**  
   Formula deterministik multi-faktor transparan (0–100) yang dapat diperkaya oleh penalaran kontekstual Asisten Kerja Google Gemma via LiteRT-LM & SQLite FTS5 BM25. Mengelompokkan tugas ke dalam 4 kategori urgensi (*Critical, High, Medium, Low*) disertai alasan logis (*explainable rationale*) dan kendali penuh pengguna (*user override* & *anti-guilt UX*).
4. **Team Collaboration & Realtime Workspaces (Pilar 4):**  
   Ruang kerja kelompok akademik berbasis Supabase Realtime WebSocket (Postgres CDC) dan PostgreSQL RLS untuk pembagian tugas, pelacakan tanggung jawab anggota tim, dan sinkronisasi status pengerjaan secara instan.
5. **Web Admin CMS & Operational Governance (Pilar 5):**  
   Pusat kendali operasional kurikulum akademik, moderasi silabus/topik belajar, pengelolaan tiket feedback pengguna, dan audit logging yang sepenuhnya terisolasi dari data privat mahasiswa.

---

## 3. Arah Teknologi & Arsitektur (Technology Stack)

### A. Android Client
- **Bahasa & Platform:** Kotlin 2.0+ pada Android Native (MinSdk 26, TargetSdk 35, CompileSdk 35).
- **UI Framework:** Jetpack Compose, Material 3, Type-Safe Navigation Compose.
- **Arsitektur Aplikasi:** Clean Architecture (Presentation, Domain, Data) dengan pola MVVM dan Unidirectional Data Flow (UDF).
- **Persistensi Lokal:** Room Database 2.6.1 (SQLite) sebagai sumber kebenaran instan (*Immediate Source of Truth*) dan Jetpack DataStore Preferences.
- **Local RAG Engine:** Tabel virtual SQLite FTS5 (`tasks_fts`) dengan algoritma pembobotan BM25 untuk pencarian leksikal cepat (<25 ms).
- **Managed BaaS Client:** Supabase via `io.github.jan-tennert.supabase:bom:3.9.0` (Auth, PostgREST, Realtime, Storage).
- **On-Device AI Engine:** Google AI Edge **LiteRT-LM** (Kotlin API) menjalankan model **Google Gemma 3 1B IT** dan **Gemma 270M**.
- **OCR Engine On-Device:** Google ML Kit Text Recognition v2 (`com.google.mlkit:text-recognition:16.0.1`).
- **Device Capability Gatekeeper:** `DeviceCapabilityManager` multi-faktor memisahkan `TextCapability` (RAM $\ge 4$ GB) vs `VisionCapability` (RAM $\ge 8$ GB).
- **Design System:** Neo-Bauhaus × Contemporary Gen Z (Inter font, grid matematis, border tegas 1.5dp, warna fungsional kontras tinggi WCAG AA, anti-guilt UX).

### B. Web Admin CMS
- **Framework & Runtime:** Next.js 15 (App Router), React 19, Node.js LTS.
- **Bahasa:** TypeScript (*strict mode*, zero `any`).
- **Styling & UI Components:** Tailwind CSS, shadcn/ui, Radix UI primitives, Lucide Icons.
- **State & Data Fetching:** React Server Components (RSC) by default, Server Actions untuk mutasi, TanStack React Query v5 untuk client-side state.
- **Supabase SSR:** `@supabase/ssr` dengan integrasi HTTP-only, secure, sameSite cookies.
- **Deployment Platform:** Vercel Hobby ($0) untuk riset/evaluasi non-komersial, dengan kesiapan kontainerisasi Docker/Coolify untuk produksi berlisensi bebas.

### C. Managed Backend & Database
- **BaaS Platform:** Supabase Managed Cloud (PostgreSQL 15+, PostgREST, GoTrue Auth, Realtime Engine, Storage).
- **Database Security:** PostgreSQL Row Level Security (RLS) di seluruh tabel `public`.
- **Target Biaya Operasional:** **Rp 0 / $0** (Beroperasi penuh dalam kuota aman Supabase Free Tier).

---

## 4. Status Rekayasa Terkini (Current Engineering Status)

| Indikator | Kondisi Terkini |
| :--- | :--- |
| **Tahapan Proyek** | `Fase 0: Fondasi Workspace & Complete Ecosystem Rebaseline (LOCKED)` |
| **Architecture Decision Records** | 41 ADR lengkap di `docs/decisions/adr/` (ADR-001 s/d ADR-041 Terkunci) |
| **Managed Cloud BaaS** | Supabase Managed BaaS via Kotlin Client v3.9.0 & Next.js `@supabase/ssr` |
| **Model AI On-Device** | Google Gemma 3 1B IT & Gemma 270M via LiteRT-LM (Opsional, Zero Cloud AI) |
| **Knowledge Intelligence Layer** | Two-Tier RAG: tasks_fts (Productivity) & academic_fts (Academic) SQLite FTS5 |
| **OCR & Image Pipeline** | Google ML Kit Text Recognition v2 + Ephemeral Cache Cleaner |
| **Web Admin CMS** | Next.js 15 App Router + React 19 + shadcn/ui + RBAC 3-Tier Terarsitektur Penuh |
| **Arsitektur Sinkronisasi** | Offline-First via Room Database + `sync_pending_queue` + WorkManager |
| **Infrastruktur Backend** | Zero Custom Backend (Go/Gin/Node/Python standalone ditiadakan) |
| **Kepatuhan Biaya** | Rp 0 / $0 (Supabase Free Tier & Vercel Hobby Guardrails strictly documented) |
| **Status Verifikasi** | `DOCUMENTED` (100% Spesifikasi) & `CORE SCAFFOLD UNIT TESTED` (41/41 Tests Lulus 100%) |

---

## 5. Panduan Navigasi Dokumentasi (Documentation Sitemap)

```text
KerjaiAja/
├── docs/
│   ├── product/
│   │   ├── PRODUCT_VISION.md           # Visi, Dual Experience (Mode A & B), 5 pilar, anti-guilt UX
│   │   ├── PRODUCT_REQUIREMENTS.md     # Spesifikasi kebutuhan fungsional & non-fungsional Android & CMS
│   │   ├── CMS_PRODUCT_REQUIREMENTS.md # Kebutuhan produk Web Admin CMS, MoSCoW & user stories
│   │   ├── ACADEMIC_KNOWLEDGE_REQUIREMENTS.md # Kebutuhan pengetahuan 12 rumpun mapel & sitasi terverifikasi
│   │   ├── CMS_KNOWLEDGE_MANAGEMENT.md # Tata kelola kurikulum CMS & alur persetujuan publikasi
│   │   ├── USER_PERSONAS.md            # Persona pengguna: Siswa, Mahasiswa, Anggota Tim, Admin
│   │   ├── USER_JOURNEYS.md            # Alur pengguna untuk produktivitas, foto soal & moderasi admin
│   │   ├── FEATURE_CATALOG.md          # Katalog fitur lengkap Android & Web Admin CMS
│   │   └── MVP_SCOPE.md                # Ruang lingkup berimbang MVP v1.0 Android + Web CMS
│   │
│   ├── architecture/
│   │   ├── SYSTEM_ARCHITECTURE.md      # Arsitektur sistem hybrid lengkap (Android + Web Admin CMS + Supabase)
│   │   ├── KNOWLEDGE_INTELLIGENCE_LAYER.md # Arsitektur terpadu Knowledge Intelligence Layer & Two-Tier RAG
│   │   ├── LOCAL_RETRIEVAL_ENGINE.md   # Mesin pencarian leksikal SQLite FTS5 BM25 & evaluasi hybrid
│   │   ├── DOCUMENT_INGESTION_PIPELINE.md # Pipeline penyerapan dokumen, validasi & intelligent chunking
│   │   ├── CONTEXT_ENGINEERING.md      # Pembangun konteks khusus, anggaran token & prompt injection defense
│   │   ├── KNOWLEDGE_SYNC.md           # Sinkronisasi delta offline-first referensi kurikulum (<100MB)
│   │   ├── RAG_SECURITY_AND_PRIVACY.md # Otorisasi pra-retrieval, isolasi data & proteksi anti-surveillance
│   │   ├── WEB_ADMIN_ARCHITECTURE.md   # Arsitektur teknis Next.js 15 App Router, React 19 & @supabase/ssr
│   │   ├── CMS_SECURITY_AND_RBAC.md    # Matriks RBAC 3-tier, admin_roles, RLS policies & audit logging
│   │   ├── CMS_DATA_ARCHITECTURE.md    # Model data bersama Android-CMS, delta sync & anti-surveillance
│   │   ├── PROGRESSIVE_AI_PROCESSING.md # Pipeline pemrosesan progresif 7 tahap (Stage 0 s/d 6)
│   │   ├── AI_MODEL_LIFECYCLE.md       # Siklus hidup model, Mutex session, auto-unload, dan LMK
│   │   ├── DEVICE_CAPABILITY_MANAGER.md # Evaluasi kelayakan hardware per-model & transparansi UI
│   │   ├── ANDROID_ARCHITECTURE.md     # Clean Architecture, MVVM, UDF, Compose di Android
│   │   ├── SUPABASE_ARCHITECTURE.md    # Spesifikasi Supabase BaaS, PostgREST, Realtime, RLS & SSR
│   │   ├── SUPABASE_FREE_TIER_AND_COST_GUARDRAILS.md # Panduan biaya Rp0 dan batas kuota Free Tier
│   │   ├── SYNC_ARCHITECTURE.md        # Arsitektur offline-first sync, sync_pending_queue & WorkManager
│   │   ├── DATA_ARCHITECTURE.md        # Skema Room SQLite hybrid, FTS5 virtual table, DataStore
│   │   ├── DEPLOYMENT_ARCHITECTURE.md  # Arsitektur penerapan zero-compute server & Play Store
│   │   ├── API_DESIGN.md               # Kontrak interface Kotlin domain & operasi Supabase client
│   │   └── BACKEND_ARCHITECTURE.md     # Arsip penghentian backend Go/Gin (SUPERSEDED)
│   │
│   ├── ai/
│   │   ├── AI_ARCHITECTURE.md          # Arsitektur on-device Gemma & LiteRT-LM runtime
│   │   ├── PHOTO_QUESTION_SOLVER_SPECIFICATION.md # Spesifikasi fitur Foto Soal & Pembahasan
│   │   ├── IMAGE_PROCESSING_ARCHITECTURE.md # Pipeline citra, downsampling, dan pembersihan efemeral
│   │   ├── OCR_STRATEGY.md             # Strategi Google ML Kit OCR v2 & human-in-the-loop review
│   │   ├── MULTIMODAL_MODEL_EVALUATION.md # Evaluasi teknis Gemma 3n E2B multimodal vision
│   │   ├── AI_OUTPUT_VALIDATION.md     # Layer validasi deterministik 7 bagian & konsistensi kalkulasi
│   │   ├── DEVICE_CAPABILITY_STRATEGY.md # Kualifikasi hardware per-model (Text vs Vision Capability)
│   │   ├── AI_ASSISTANT_SPECIFICATION.md # Spesifikasi Asisten Kerja 5 use-case & larangan chatbot
│   │   ├── ON_DEVICE_MODEL_EVALUATION.md # Evaluasi komparatif Gemma 3 1B IT & Gemma 270M
│   │   ├── LOCAL_RAG_ARCHITECTURE.md   # Spesifikasi Local RAG via SQLite FTS5 BM25
│   │   ├── MODEL_STRATEGY.md           # Strategi model on-device & matriks sumber daya ponsel
│   │   └── AI_EVALUATION_STRATEGY.md   # KerjaiAja On-Device Benchmark Suite 11 dimensi
│   │
│   ├── deployment/
│   │   └── VERCEL_SUPABASE_DEPLOYMENT.md # Evaluasi Vercel Hobby ($0), fallback Docker/Coolify & env security
│   │
│   ├── quality/
│   │   ├── SUBJECT_BENCHMARK.md        # Standar benchmark 12 rumpun mata pelajaran
│   │   ├── OCR_EVALUATION.md           # Uji ketahanan OCR pada 8 variasi citra lapangan
│   │   └── DEVICE_BENCHMARK.md         # Profiling hardware fisik (Low-End, Mid-Range, High-End)
│   │
│   ├── performance/
│   │   ├── AI_PERFORMANCE_BENCHMARK.md # Rencana benchmark performa AI on-device reproducible
│   │   ├── APP_SIZE_AND_RESOURCE_BUDGET.md # Anggaran ukuran AAB (<20 MB), RAM, & baterai
│   │   └── RAG_BENCHMARK.md            # Profiling latensi kueri, alokasi RAM, CPU & baterai FTS5
│   │
│   ├── testing/
│   │   ├── PHOTO_QUESTION_SOLVER_TEST_PLAN.md # Rencana pengujian komprehensif solver foto soal
│   │   ├── CMS_TEST_STRATEGY.md        # Piramida pengujian CMS (Vitest, RLS integration, Playwright E2E)
│   │   └── RAG_EVALUATION_PLAN.md      # Evaluasi retrieval P@K, R@K, MRR & Golden Test Suite
│   │
│   ├── design/
│   │   ├── DESIGN_SYSTEM.md            # Token Neo-Bauhaus, palet warna, tipografi Inter, grid
│   │   └── DESIGN_DIRECTION.md         # Filosofi visual & prinsip form follows function
│   │
│   ├── security/
│   │   ├── SECURITY_ARCHITECTURE.md    # Threat model, anon key vs service key, Keystore, OWASP MASVS
│   │   ├── SECURITY_REVIEW.md          # Tinjauan keamanan komprehensif OWASP MASVS v2.0
│   │   ├── PRIVACY_REVIEW.md           # Tinjauan privasi data citra efemeral & UU PDP
│   │   └── PLAY_STORE_READINESS.md     # Target SDK 35, Data Safety, Account Deletion compliance
│   │
│   ├── development/
│   │   ├── DEVELOPMENT_ROADMAP.md      # Peta jalan implementasi 8 fase Android & Web CMS
│   │   ├── CODING_STANDARDS.md         # Standar kode Kotlin Jetpack Compose & TypeScript
│   │   └── TESTING_STRATEGY.md         # Piramida pengujian otomatis Android Native
│   │
│   ├── decisions/
│   │   ├── ARCHITECTURE_DECISIONS.md   # Master ADR index (ADR-001 s/d ADR-041 lengkap)
│   │   └── adr/                        # Berkas individual ADR-001 hingga ADR-041
│   │
│   └── audits/
│       ├── FINAL_READINESS_REPORT.md          # Laporan kesiapan rekayasa akhir & status kejujuran
│       ├── ARCHITECTURE_MIGRATION_LOG.md      # Log kronologis keputusan migrasi arsitektur (Part 1-7)
│       ├── HYBRID_ARCHITECTURE_REBASELINE_AUDIT.md # Laporan audit rebaseline arsitektur hybrid
│       ├── ON_DEVICE_AI_ARCHITECTURE_AUDIT.md # Laporan audit rebaseline on-device sebelumnya
│       ├── SUPERPOWERS_DOCUMENTATION_AUDIT.md # Laporan audit historis lama (Preserved)
│       └── SUPERPOWERS_DOCUMENTATION_REVISION_LOG.md # Log revisi audit lama (Preserved)
│
├── planning/
│   ├── BACKLOG.md                      # Backlog modular (TASK, SUPA, SYNC, PRIO, DEV, ASST, TEAM, DES, SEC, SOLV, CMS, KNOW)
│   └── IMPLEMENTATION_PHASES.md        # Rencana aksi eksekusi teknis 8 fase implementasi
│
└── gradle/
    └── libs.versions.toml              # Gradle version catalog resmi Android (Supabase BOM 3.9.0 & ML Kit 16.0.1)
```

