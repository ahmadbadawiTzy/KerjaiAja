# Rencana Fase Implementasi (Implementation Phases) — KerjaiAja
## Hybrid Offline-First Platform · Supabase Managed BaaS · On-Device AI · Photo Question Solver

Dokumen ini mendefinisikan rincian operasional teknis untuk 8 tahapan implementasi **KerjaiAja** pasca-final rebaseline arsitektur hybrid dengan fitur terintegrasi **Foto Soal & Pembahasan**.

---

## Tahap 0: Fondasi Workspace, Final Rebaseline Arsitektur & ADR Lock (Status: SELESAI / DOCUMENTED & UNIT TESTED)
- **Tujuan:** Mengunci keputusan arsitektur hybrid komprehensif (ADR-001 s/d ADR-041), mendokumentasikan spesifikasi Supabase BaaS, offline-first sync, guardrail biaya Rp0 Free Tier, evaluasi hardware per-model, arsitektur Progressive AI Processing 7 tahap, siklus hidup model & memori, arsitektur Web Admin CMS Next.js 15, skema database bersama, arsitektur Knowledge Intelligence Layer (Two-Tier RAG, Ingestion, Intelligent Chunking, Context Engineering, Knowledge Sync), serta unit test scaffold tervalidasi.
- **Aktivitas:**
  1. Penulisan dan verifikasi ADR-001 hingga ADR-041 di `docs/decisions/adr/` serta pembaruan master index `docs/decisions/ARCHITECTURE_DECISIONS.md`.
  2. Penyusunan dokumen arsitektur dan spesifikasi baru:
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
     - `docs/product/CMS_PRODUCT_REQUIREMENTS.md`
     - `docs/architecture/WEB_ADMIN_ARCHITECTURE.md`
     - `docs/architecture/CMS_SECURITY_AND_RBAC.md`
     - `docs/architecture/CMS_DATA_ARCHITECTURE.md`
     - `docs/deployment/VERCEL_SUPABASE_DEPLOYMENT.md`
     - `docs/testing/CMS_TEST_STRATEGY.md`
     - `docs/architecture/PROGRESSIVE_AI_PROCESSING.md`
     - `docs/architecture/AI_MODEL_LIFECYCLE.md`
     - `docs/architecture/DEVICE_CAPABILITY_MANAGER.md`
     - `docs/performance/AI_PERFORMANCE_BENCHMARK.md`
     - `docs/testing/PHOTO_QUESTION_SOLVER_TEST_PLAN.md`
     - `docs/ai/IMAGE_PROCESSING_ARCHITECTURE.md`
     - `docs/ai/OCR_STRATEGY.md`
     - `docs/ai/MULTIMODAL_MODEL_EVALUATION.md`
     - `docs/ai/AI_OUTPUT_VALIDATION.md`
     - `docs/quality/SUBJECT_BENCHMARK.md`
     - `docs/quality/OCR_EVALUATION.md`
     - `docs/quality/DEVICE_BENCHMARK.md`
     - `docs/security/PRIVACY_REVIEW.md`
     - `docs/security/SECURITY_REVIEW.md`
  3. Pembaruan seluruh dokumen produk, arsitektur sistem, data, roadmap, testing, dan backlog.
  4. Penyiapan dependensi `gradle/libs.versions.toml` (Supabase BOM 3.9.0, Ktor 3.0.3, ML Kit Text Recognition 16.0.1).
- **Kriteria Selesai (DoD):** Seluruh dokumen konsisten, zero custom backend server, zero external AI API, arsitektur Progressive AI Processing Pipeline, Web Admin CMS, Knowledge Intelligence Layer, dan lifecycle memori tervalidasi dengan unit test nyata, dan struktur siap untuk implementasi kode.

---

## Tahap 1: Inisialisasi Android Core, Room & Supabase Client (Estimasi: Sprint 1)
- **Tujuan:** Membangun fondasi persistensi lokal berkecepatan tinggi, antarmuka Neo-Bauhaus, dan inisialisasi Supabase SDK.
- **Aktivitas:**
  1. Setup struktur paket Clean Architecture Android (`core/`, `ai/`, `data/`, `domain/`, `presentation/`).
  2. Implementasi theme dan komponen dasar Neo-Bauhaus di `core/designsystem`.
  3. Konfigurasi Room Database 2.6.1 (`TaskEntity`, `SubjectEntity`, `SubtaskEntity`, `SyncQueueEntity`) beserta DAO reaktif berbasis Kotlin `Flow`.
  4. Implementasi tabel virtual SQLite FTS5 (`tasks_fts`) dengan triggers otomatis untuk pencarian leksikal cepat.
  5. Konfigurasi `SupabaseClient` via Hilt (`auth-kt`, `postgrest-kt`, `realtime-kt`, `storage-kt`) dan implementasi alur Guest-First Onboarding.
  6. Pembangunan layar Compose: Dashboard Tugas, Tambah/Edit Tugas, dan Jadwal Kuliah.
- **Kriteria Selesai (DoD):** Mahasiswa dapat mencatat tugas seketika secara offline (Guest Mode) dengan UI Neo-Bauhaus responsif (60/120 FPS).

---

## Tahap 2: Offline-First Sync, Deterministic Priority & Team Realtime (Estimasi: Sprint 2)
- **Tujuan:** Mengaktifkan sinkronisasi background, formula prioritas deterministik, dan kolaborasi tugas tim.
- **Aktivitas:**
  1. Implementasi `DeterministicPriorityEngine`: formula matematika 4 faktor (0–100) dan klasifikasi 4 kategori urgensi instan di CPU.
  2. Implementasi `SyncWorker` (WorkManager) untuk memproses mutasi `sync_pending_queue` ke Supabase PostgREST dengan resolusi konflik Last-Write-Wins (LWW) dan soft-deletion tombstone.
  3. Implementasi alur migrasi akun tamu ke akun cloud Supabase secara transparan.
  4. Pembangunan modul Tim: pembuatan proyek kelompok, pembagian peran, proteksi PostgreSQL RLS, dan sinkronisasi status tugas via Supabase Realtime WebSocket.
  5. Implementasi Anti-Guilt UX untuk tugas melewati deadline (*overdue*).
- **Kriteria Selesai (DoD):** Sinkronisasi background berjalan mulus tanpa memblokir UI, skor prioritas dihitung instan (<5 ms), dan perubahan tugas kelompok tersinkronisasi real-time antar-anggota tim.

---

## Tahap 3: Device Capability Management & On-Device Gemma Assistant (Estimasi: Sprint 3)
- **Tujuan:** Mengintegrasikan Asisten Kerja on-device Google Gemma via LiteRT-LM yang diproteksi kualifikasi hardware.
- **Aktivitas:**
  1. Implementasi `DeviceCapabilityManager`: evaluasi multi-faktor RAM bebas ($\ge 1.5\text{ GB}$), arsitektur 64-bit ABI, storage internal ($\ge 3.0\text{ GB}$), thermal status, dan level baterai.
  2. Integrasi SDK Google AI Edge **LiteRT-LM** melalui abstraksi `AiEngine` (`Gemma 3 1B IT` & `Gemma 270M`).
  3. Implementasi `LocalRagRetriever`: ekstraksi konteks tugas relevan dari `tasks_fts` (BM25) dengan pruning token budget ($\le 800$ token).
  4. Implementasi 5 use-case Asisten Kerja: *Ask About My Work*, *Task Breakdown*, *Smart Planning*, *Contextual Recommendations*, *Work Review*.
  5. `StructuredOutputValidator`: validasi parsing JSON via `kotlinx.serialization` dengan regex auto-repair dan fallback deterministik.
  6. `ModelDownloadManager`: unduhan model on-demand dari CDN resmi Google AI Edge / Hugging Face dengan verifikasi SHA-256 dan hak hapus instan.
- **Kriteria Selesai (DoD):** Asisten Kerja mampu merespons tugas kontekstual dalam waktu $< 3.5$ detik di ponsel secara offline, zero byte data tugas yang keluar dari perangkat, dan zero LMK crash.

---

## Tahap 4: Fitur Foto Soal & Pembahasan — Jalur A (OCR-First) (Estimasi: Sprint 4)
- **Tujuan:** Mewujudkan fitur pemecah soal akademik melalui integrasi kamera, OCR lokal, review siswa, inferensi teks, dan validasi output 7 bagian.
- **Aktivitas:**
  1. Implementasi modul penangkap citra menggunakan Jetpack CameraX dan Android Photo Picker dengan batas memori piksel ($\le 1920\times 1080$).
  2. Integrasi Google ML Kit Text Recognition v2 dengan penghitungan confidence score per kata dan threshold peringatan.
  3. Pembuatan antarmuka *Human-in-the-Loop Review* dengan toolbar simbol akademik (matematika/sains) dan pemilih 12 rumpun mata pelajaran.
  4. Implementasi `AnswerValidator`: validasi deterministik 7 bagian, integritas perhitungan aritmetika, pengecekan satuan, dan penanda ketidakpastian (*uncertaintyFlag*).
  5. Mekanisme pembersihan efemeral otomatis berkas citra dari `context.cacheDir`.
- **Kriteria Selesai (DoD):** Siswa dapat memfoto soal cetak, mengoreksi teks dalam review screen, mendapatkan pembahasan terstruktur 7 bagian dalam waktu $< 5$ detik, dengan jaminan 0 byte citra diunggah ke cloud.

---

## Tahap 5: Evaluasi Multimodal Vision Model — Jalur B (Lab & High-End) (Estimasi: Sprint 5)
- **Tujuan:** Mengeksplorasi kemampuan inferensi citra langsung on-device untuk soal diagram geometri dan grafik sains.
- **Aktivitas:**
  1. Implementasi gerbang evaluasi `VisionCapability`: pengecekan syarat RAM $\ge 8\text{ GB}$, RAM bebas $\ge 3.0\text{ GB}$, storage bebas $\ge 5\text{ GB}$, dan NPU $\ge 15\text{ TOPS}$.
  2. Penyiapan pipeline evaluasi model multimodal **Gemma 3n E2B** via LiteRT runtime.
  3. Manajemen paket unduhan model visual terpisah (*opt-in*).
  4. Eksekusi pengujian termal, memori RSS, dan latensi inferensi diagram.
- **Kriteria Selesai (DoD):** Tersedianya laporan lab benchmark yang objektif mengenai kelayakan model visual on-device pada perangkat flagship.

---

## Tahap 6: Comprehensive Quality & Robustness Benchmarking (Estimasi: Sprint 6)
- **Tujuan:** Melakukan verifikasi mutu berulang pada 12 rumpun mata pelajaran, ketahanan OCR lapangan, dan stabilitas perangkat.
- **Aktivitas:**
  1. Eksekusi suite pengujian 240 soal pada 12 rumpun mata pelajaran (`docs/quality/SUBJECT_BENCHMARK.md`).
  2. Pengujian ketahanan OCR terhadap 8 kondisi citra lapangan (blur, low light, glare, tulisan tangan).
  3. Profiling performa fisik pada perangkat Low-End, Mid-Range, dan High-End (`docs/quality/DEVICE_BENCHMARK.md`).
  4. Verifikasi ketahanan memori terhadap *LowMemoryKiller (LMK)*.
- **Kriteria Selesai (DoD):** Akurasi konsep $\ge 85\%$ pada mata pelajaran eksakta, 0% crash UI, dan kelengkapan 7 bagian solusi $= 100\%$.

---

## Tahap 7: Production Hardening, Security/Privacy Audit & Play Store Release (Estimasi: Sprint 7)
- **Tujuan:** Menjamin keandalan sistem, audit keamanan OWASP MASVS, kepatuhan Google Play Store (Target SDK 35), dan produksi artefak rilis.
- **Aktivitas:**
  1. Eksekusi suite pengujian otomatis lengkap (Unit Test $\ge 70\%$, Widget Test $\ge 20\%$, Integration Test $\ge 10\%$).
  2. Audit keamanan MASVS: penyimpanan token Keystore (AES-256-GCM), penegakan RLS Supabase, dan network security config.
  3. Audit privasi citra: verifikasi tidak adanya foto soal di Supabase Storage maupun database Room.
  4. Verifikasi kepatuhan batasan kuota Supabase Free Tier ($0).
  5. Konfigurasi R8 release build shrinking (target ukuran instalasi AAB $\le 20\text{ MB}$).
  6. Penyusunan data safety form dan rilis AAB ke Google Play Console.
- **Kriteria Selesai (DoD):** Laporan audit keamanan bebas temuan high/critical, test pass rate 100%, dan rilis siap untuk ditinjau oleh tim Google Play Store.
