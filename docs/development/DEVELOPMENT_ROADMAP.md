# Roadmap Pengembangan (Development Roadmap) — KerjaiAja
## Hybrid Offline-First Platform · Supabase Managed BaaS · On-Device AI · Photo Question Solver

Dokumen ini menjabarkan peta jalan pengembangan aplikasi **KerjaiAja** dari tahap inisialisasi hingga rilis produksi Google Play Store pasca-final rebaseline arsitektur hybrid dengan fitur terintegrasi **Foto Soal & Pembahasan**.

---

## 1. Ikhtisar 8 Tahapan Implementasi (Phases Overview)

```text
[Fase 0: Fondasi & Final Rebaseline] ➡️ [Fase 1: Android Core, Room & Supabase] ➡️ [Fase 2: Sync Engine & Deterministic Priority]
                                                                                                    ⬇️
[Fase 5: Vision Model Lab (Jalur B)] ⬅️ [Fase 4: Foto Soal & Pembahasan (Jalur A)] ⬅️ [Fase 3: Device Capability & Gemma Text AI]
          ⬇️
[Fase 6: Subject & Quality Benchmarks] ➡️ [Fase 7: Hardening, Security Audit & Play Store Release]
```

---

## 2. Rincian 8 Fase Kerja

### Fase 0: Fondasi Workspace, Final Rebaseline Arsitektur & ADR Lock (Status: SELESAI / DOCUMENTED & UNIT TESTED)
- **Target:** Mengunci keputusan arsitektur hybrid komprehensif (ADR-001 s/d ADR-038), mendokumentasikan spesifikasi Supabase Managed BaaS, offline-first sync, batasan biaya Rp0 Free Tier, pemisahan kapabilitas teks vs visual, fitur Foto Soal & Pembahasan 7 bagian, Progressive AI Processing Pipeline (7 tahap), tata kelola memori model on-device, arsitektur Web Admin CMS Next.js 15, skema database bersama, dan verifikasi 41 unit tests scaffold.
- **Deliverables:**
  - 38 dokumen ADR lengkap di `docs/decisions/adr/` (ADR-001 s/d ADR-038).
  - Spesifikasi Web Admin CMS di `docs/architecture/WEB_ADMIN_ARCHITECTURE.md`.
  - Spesifikasi CMS Security & RBAC di `docs/architecture/CMS_SECURITY_AND_RBAC.md`.
  - Spesifikasi Data Bersama di `docs/architecture/CMS_DATA_ARCHITECTURE.md`.
  - Strategi Deployment Vercel & Supabase di `docs/deployment/VERCEL_SUPABASE_DEPLOYMENT.md`.
  - Strategi Pengujian CMS di `docs/testing/CMS_TEST_STRATEGY.md`.
  - Spesifikasi Progressive AI Processing di `docs/architecture/PROGRESSIVE_AI_PROCESSING.md`.
  - Scaffold inti dan 41 unit tests Kotlin yang tervalidasi 100% lulus di `app/src/test/`.
  - Katalog dependensi resmi di `gradle/libs.versions.toml`.

### Fase 1: Klien Android Core, Room Database & Supabase Integration (Sprint 1)
- **Target:** Membangun fondasi Android Native murni (Kotlin 2.0+, Jetpack Compose Neo-Bauhaus, Room 2.6.1, Hilt) dan inisialisasi Supabase Kotlin Client 3.9.0.
- **Deliverables:**
  - Setup Room Database lokal (`TaskEntity`, `SubjectEntity`, `SubtaskEntity`, `SyncQueueEntity`).
  - Implementasi tabel virtual SQLite FTS5 (`tasks_fts`) dengan triggers otomatis.
  - Setup Supabase Client (`auth-kt`, `postgrest-kt`, `realtime-kt`, `storage-kt`) via Ktor Android Engine.
  - Implementasi alur **Guest-First Onboarding** dan migrasi akun transparan ke Supabase Auth.
  - Tampilan navigasi Compose: Dashboard Tugas, Tambah/Edit Tugas, Manajemen Mata Pelajaran, Jadwal.

### Fase 2: Offline-First Sync Engine, Deterministic Priority & Team Collaboration (Sprint 2)
- **Target:** Realisasi sinkronisasi data background, formula deterministik prioritas 4 faktor, dan kolaborasi tugas tim.
- **Deliverables:**
  - `DeterministicPriorityEngine`: Formula skor prioritas 4 faktor (0–100) dan klasifikasi 4 tingkatan urgensi instan di CPU.
  - `SyncManager` & `SyncWorker` (WorkManager): Pemrosesan batch `sync_pending_queue` ke Supabase PostgREST dengan penanganan konflik LWW (Last-Write-Wins).
  - Ruang Kerja Tim: Kolaborasi kelompok dengan proteksi PostgreSQL RLS dan pembaruan instan via Supabase Realtime WebSocket.
  - Antarmuka *Anti-Guilt UX*: Opsi adaptif untuk tugas overdue.

### Fase 3: Device Capability Management & On-Device Gemma Text Assistant (Sprint 3)
- **Target:** Evaluasi hardware cerdas dan integrasi Asisten Kerja on-device Google Gemma (1B IT / 270M) via LiteRT-LM.
- **Deliverables:**
  - `DeviceCapabilityManager`: Evaluasi multi-faktor terpisah untuk `TextCapability` vs `VisionCapability`.
  - Integrasi SDK Google AI Edge **LiteRT-LM** melalui abstraksi `AiEngine`.
  - Implementasi `LocalRagRetriever`: Ekstraksi konteks tugas relevan dari `tasks_fts` (BM25).
  - Implementasi 5 skenario Asisten Kerja: *Ask About My Work*, *Task Breakdown*, *Smart Planning*, *Contextual Recommendations*, *Work Review*.
  - `ModelDownloadManager`: Manajemen unduhan model teks terverifikasi (SHA-256) dengan opsi hapus instan.

### Fase 4: Fitur Foto Soal & Pembahasan — Jalur A (OCR-First) (Sprint 4)
- **Target:** Implementasi fitur pemecah soal akademik via kamera/galeri, OCR lokal, review pengguna, inferensi teks, dan validasi output deterministik.
- **Deliverables:**
  - Integrasi Jetpack CameraX & Android Photo Picker dengan panduan bingkai dokumen.
  - Integrasi Google ML Kit Text Recognition v2 (`com.google.mlkit:text-recognition:16.0.1`).
  - Layar *Human-in-the-Loop Review* dengan toolbar simbol akademik matematika/sains.
  - Selector 12 Rumpun Mata Pelajaran untuk injeksi konteks *prompt*.
  - `AnswerValidator`: Validasi deterministik format 7 bagian, konsistensi aritmetika, dan pengecekan satuan.
  - Pembersihan berkas citra efemeral seketika dari `context.cacheDir`.

### Fase 5: Evaluasi Multimodal Vision Model — Jalur B (Lab & High-End) (Sprint 5)
- **Target:** Eksplorasi inferensi visual langsung on-device untuk soal diagram geometri, grafik kurva, dan ilustrasi sains.
- **Deliverables:**
  - Gerbang seleksi ketat `VisionCapability` (RAM $\ge 8\text{ GB}$, NPU $\ge 15\text{ TOPS}$, storage $\ge 5\text{ GB}$).
  - Pipeline benchmarking model multimodal **Gemma 3n E2B** via LiteRT runtime.
  - Manajemen download model visual terpisah (*explicit opt-in*).
  - Protokol evaluasi termal, konsumsi memori, dan latensi inferensi citra.

### Fase 6: Comprehensive Quality & Robustness Benchmarking (Sprint 6)
- **Target:** Pengujian mutu akademik berulang, evaluasi ketahanan OCR lapangan, dan profiling perangkat fisik.
- **Deliverables:**
  - Uji dataset 240 soal akademik pada 12 rumpun mata pelajaran (`docs/quality/SUBJECT_BENCHMARK.md`).
  - Uji ketahanan OCR pada 8 skenario citra lapangan (blur, low light, glare, miring, tulisan tangan).
  - Profiling performa pada perangkat Low-End, Mid-Range, dan High-End (`docs/quality/DEVICE_BENCHMARK.md`).
  - Verifikasi ketiadaan memory leak dan stabilitas LMK.

### Fase 7: Production Hardening, Security/Privacy Audit & Play Store Release (Sprint 7)
- **Target:** Audit keamanan menyeluruh, verifikasi regulasi Google Play & UU PDP, optimasi binary R8, dan rilis AAB.
- **Deliverables:**
  - Audit OWASP MASVS (penyimpanan token Keystore, RLS Supabase, network security config).
  - Audit privasi citra efemeral (verifikasi 0 byte citra diunggah ke cloud).
  - Verifikasi kepatuhan batasan gratis Supabase Free Tier ($0).
  - Konfigurasi R8 release shrinking (target ukuran APK dasar $\le 20\text{ MB}$).
  - Penerbitan Android App Bundle (AAB) ke Google Play Console (Track Internal/Closed Testing).
