# Status Proyek KerjaiAja
## Hybrid Offline-First Platform · Supabase Managed BaaS · On-Device AI · Photo Question Solver

> **Status Saat Ini:** `Phase 1 & Phase 2 — Core Implementation, Build & Verification (VERIFIED)`  
> **Terakhir Diperbarui:** 2026-11-20 (Sesi 3 - Penutupan Celah Rebaseline)  
> **Kondisi Implementasi:** *Fondasi arsitektur hybrid (Dual Experience Mode A & Mode B), TaskList UI terhubung Room nyata via TaskViewModel + Hilt DI, Room persistence across reopen terbukti 100%, SupabaseTaskRemoteDataSource via PostgREST SDK resmi, SyncScheduler (WorkManager 15m + immediate), Skema SQL Supabase & RLS Canonical + Teams di supabase/migrations/, Pengujian RLS nyata di PostgreSQL 16 Docker (8/8 skenario PASS), Secret scan 0 temuan, ABI Filtering arm64-v8a aktif (APK release 27 MB, AAB 17 MB), Lint 0 errors, 74/74 unit test Kotlin lulus 100% across 16 classes, Web Admin CMS Next.js 15 sukses di-build 100%, Fitur Foto Soal/OCR, Vision Lab, dan Web Admin CMS berstatus PASCA-MVP / DITAHAN MENUNGGU KEPUTUSAN PENGGUNA.*

---

## 1. Ringkasan Status Arsitektur Terkunci

Proyek **KerjaiAja** telah mengunci arsitektur definitif: **Hybrid Offline-First Productivity Platform: Supabase Managed BaaS + Optional On-Device AI + Photo Question Solver**.

Poin-poin arsitektural utama yang disahkan:
1. **Dual Experience Product (ADR-023):**
   - *Mode A (Produktivitas Penuh Tanpa AI):* 100% fungsional, inklusif untuk semua pengguna tanpa perlu model AI.
   - *Mode B (Produktivitas dengan Asisten Belajar & Foto Soal Opsional):* Fitur analitik kontekstual dan pemecah soal foto untuk perangkat yang memenuhi kualifikasi hardware.
2. **Foto Soal & Pembahasan (ADR-027 s/d ADR-032):**
   - *Status:* **PASCA-MVP / DITAHAN MENUNGGU KEPUTUSAN PENGGUNA**. Tidak dikembangkan lebih jauh dalam MVP dasar sampai ada instruksi eksplisit pengguna.
   - *Jalur A (OCR-First - Universal):* Google ML Kit Text Recognition v2 + Human-in-the-Loop Review Screen + Gemma 3 1B/270M Text Model + Answer Validation Layer (format solusi terstruktur 7 bagian).
   - *Jalur B (Laboratorium Multimodal - Khusus Perangkat Tinggi):* Evaluasi eksperimental model vision **Gemma 3n E2B** untuk soal diagram geometri/grafik sains (RAM $\ge 8$ GB, NPU $\ge 15$ TOPS).
   - *Privasi Efemeral:* Citra hanya diproses lokal di `cacheDir`, zero cloud image upload, penghapusan file seketika.
   - *12 Rumpun Mata Pelajaran:* Standar evaluasi kurikulum akademik menyeluruh.
3. **Supabase Managed BaaS via Kotlin SDK (ADR-020):** Menggunakan pustaka resmi `io.github.jan-tennert.supabase:bom:3.9.0` (Auth, PostgREST, Realtime, Storage) via Ktor Android Engine. Dilarang membuat custom backend (Go/Gin/Node/Python ditiadakan).
4. **Guest-First & Seamless Migration (ADR-021):** Pengguna dapat langsung memakai aplikasi tanpa registrasi; migrasi data transparan saat mendaftar akun Supabase.
5. **Offline-First Synchronization (ADR-022):** Prinsip local-write-first di Room Database, antrean `sync_pending_queue`, dan resolusi konflik Last-Write-Wins (LWW) dengan soft-deletion tombstone (`deleted_at`).
6. **Device Capability Management (ADR-024 & ADR-029):** Evaluasi hardware terpisah: `TextCapability` (RAM $\ge 4$ GB) vs `VisionCapability` (RAM $\ge 8$ GB).
7. **Asisten Kerja Kontekstual & Chatbot Ban (ADR-025):** AI dibatasi pada 5 skenario kerja terstruktur; generic chatbot dilarang keras.
8. **Team Collaboration via Supabase Realtime & RLS (ADR-026):** Kolaborasi tugas kelompok diproteksi oleh PostgreSQL Row Level Security dan disinkronkan via WebSocket CDC.
9. **Kebijakan Biaya Rp0 (Supabase Free Tier Guardrails):** Beroperasi penuh dalam batas aman Free Plan ($0). Skrip anti-pausing manipulatif dilarang. Model weights AI dilarang disimpan di Supabase Storage.
10. **Progressive AI Processing Architecture (ADR-033):** Pipeline progresif 7 tahap (Stage 0 s/d Stage 6) dengan doktrin: *"Lakukan sebanyak mungkin dengan pemrosesan deterministik ringan sebelum memanggil model AI."* Menyelesaikan $>90\%$ aksi tanpa LLM.
11. **On-Device Model Lifecycle & Memory Management (ADR-034):** Lazy loading, proteksi single active session `Mutex`, batas timeout 15 detik, timer idle auto-unload 5 menit, dan proteksi LMK `onTrimMemory(15)`.
12. **Per-Model Device Capability Evaluation (ADR-035):** Evaluasi granular kelayakan perangkat per-kemampuan dan per-model (`ML Kit OCR`, `Gemma 270M`, `Gemma 1B IT`, `Gemma 3n E2B`) dilengkapi dialog transparansi spesifikasi hardware.
13. **Web Admin CMS Architecture (ADR-036):** Status: **PASCA-MVP / DITAHAN MENUNGGU KEPUTUSAN PENGGUNA**. Scaffold Next.js 15 App Router, React 19, TypeScript strict mode, Tailwind CSS, shadcn/ui, dan `@supabase/ssr` siap.
14. **CMS Role-Based Access Control (ADR-037):** Model otorisasi 3 tingkat (`SUPER_ADMIN`, `ADMIN`, `SUPPORT`), penegakan database RLS via `is_admin()`, tanpa registrasi mandiri publik (invitation-only), isolasi service role key, dan jejak audit otomatis pada `admin_audit_logs`.
15. **Shared Database Schema & CMS Multi-Tenant RLS (ADR-038):** Integrasi skema database Supabase PostgreSQL bersama antara Android dan CMS, menjaga alur Delta Sync (`academic_references`), isolasi data lokal Room, dan kepatuhan guardrails Free Tier ($0).
16. **Two-Tier Knowledge Intelligence Layer (ADR-039):** Pemisahan fisik & logis antara domain *Productivity Knowledge* (`tasks_fts`) dan *Academic Knowledge* (`academic_fts`) menggunakan baseline SQLite FTS5 BM25 ringan (<15 MB RAM, <20 ms latensi) tanpa model embedding berat.
17. **Document Ingestion & Intelligent Chunking (ADR-040):** Segmentasi cerdas berorientasi struktur yang menjaga integritas rumus matematika LaTeX dan header tabel, hash SHA-256 anti-duplikasi, serta sitasi terverifikasi (nama buku, bab, nomor halaman).
18. **Context Engineering & Offline Knowledge Sync (ADR-041):** Pembatasan anggaran token ketat (Productivity $\le 800$, Academic $\le 1200$), pertahanan Prompt Injection (data pasif di `<retrieved_references>`), serta sinkronisasi delta luring (<100 MB budget) dengan kontrol penuh pengguna.

---

## 2. Matriks Status Kesiapan Rekayasa

| Area Rekayasa | Status | Catatan / Artefak |
| :--- | :---: | :--- |
| **Workspace & Struktur Direktori** | `DOCUMENTED` | Direktori root terisolasi di `Android/KerjaiAja/` |
| **Spesifikasi Produk (Dual Exp, Solver, CMS, RAG)** | `DOCUMENTED` | Visi, PRD, Personas, Journeys, Catalog, Scope, `CMS_PRODUCT_REQUIREMENTS.md`, `ACADEMIC_KNOWLEDGE_REQUIREMENTS.md` |
| **Catatan Keputusan Arsitektur** | `DOCUMENTED` | 41 ADR lengkap di `docs/decisions/adr/` (ADR-001 s/d ADR-041 terkunci) |
| **Android Version Catalog** | `VERIFIED` | `gradle/libs.versions.toml` (Supabase 3.9.0, ML Kit OCR 16.0.1, Room 2.6.1, LiteRT, WorkManager 2.9.1) |
| **Scaffold Logika Inti & Unit Tests** | `VERIFIED` | **74 unit tests across 16 classes lulus 100%** di `app/src/test/` (ViewModel, SyncEngine, Priority, RAG, Device, Router, Lifecycle, Solver, Validator, SyncScheduler, RoomPersistenceReopen) |
| **Room Database & Hilt DI** | `VERIFIED` | `DatabaseModule.kt`, `RepositoryModule.kt`, `TaskViewModel.kt` terhubung Room via StateFlow |
| **Room Persistence Across Reopen** | `VERIFIED` | `RoomPersistenceReopenTest.kt` membuktikan data persist 100% setelah db ditutup dan dibuka kembali via Robolectric JVM |
| **Offline-First Sync Engine & Worker** | `VERIFIED` | `SyncEngine.kt`, `SyncWorker.kt`, `SyncScheduler.kt`, `SupabaseTaskRemoteDataSource.kt` via PostgREST SDK resmi |
| **Supabase Migrations & RLS Isolation** | `VERIFIED (LOCAL POSTGRES 16)` | `20261120000001_create_kerjaiaja_canonical_schema.sql` & `20261120000002_create_teams_and_collaboration_schema.sql` teruji 8/8 PASS di Postgres 16 Docker |
| **Supabase Cloud Remote Deployment** | `BLOCKED` | Menunggu pembuatan/linking instance cloud dari pengguna |
| **Secret Scan Codebase** | `VERIFIED` | 0 temuan hardcoded secret di `app/`, `web-admin/`, dan `supabase/` |
| **Android Release Binary & Size Optimization** | `VERIFIED` | `./gradlew assembleRelease` (APK 27 MB, arm64-v8a ter-filter, R8 Minified) & `./gradlew bundleRelease` (AAB 17 MB, di bawah budget $\le 25$ MB) |
| **Android Lint Quality Gate** | `VERIFIED` | `./gradlew lintDebug`: **0 errors, 67 warnings** (2 disabled checks terbukti bug resmi AGP 8.7.2 / Kotlin 2.0 IssueTracker #343606775 & #342938810) |
| **Foto Soal/OCR & Vision Lab** | `PASCA-MVP / DITAHAN` | Ditahan menunggu keputusan pengguna; tidak dikembangkan lebih jauh |
| **Web Admin CMS (Next.js 15)** | `PASCA-MVP / DITAHAN` | Ditahan menunggu keputusan pengguna; scaffold, tsc (0 errors), lint (0 errors), build (6 pages) lulus 100% |

---

## 3. Langkah Kerja Berikutnya & Keputusan yang Ditunggu dari Pengguna

1. **Keputusan Lingkup Lanjutan:**
   - Menunggu persetujuan pengguna untuk melanjutkan atau tetap menahan fitur: Foto Soal/OCR, Vision Multimodal Lab, dan Web Admin CMS.
2. **Deploy Skema Supabase ke Project Cloud:**
   - Jalankan migrasi canonical dan teams pada remote instance Supabase begitu project URL dan anon key disiapkan oleh user.


