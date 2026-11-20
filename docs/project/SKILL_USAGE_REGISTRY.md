# Skill Usage Registry — KerjaiAja
## Multi-Agent Skill-Driven Collaborative Engineering System

Dokumen ini adalah *living skill usage registry* resmi proyek **KerjaiAja**. Registry ini mendokumentasikan hasil *skill discovery*, pemetaan agen, instruksi yang dibaca, alasan pemilihan, cara aplikasi pada kode produksi, bukti verifikasi nyata, serta batasan teknis dari setiap skill yang terpasang di ekosistem Antigravity.

**Terakhir Diperbarui:** 2026-11-20  
**Status Audit Skill:** TERVERIFIKASI & AKTIF  
**Prinsip Utama:** Bebas dari skill fiktif, berbasis lokasi nyata pada sistem, difokuskan pada hasil kode produksi nyata (*production-ready code*).

---

## 1. Skill Discovery & Assignment Matrix

Seluruh skill yang tercantum di bawah ini telah diverifikasi keberadaannya secara fisik pada *filesystem* host (`/home/ahmadbadawi/`):

| Agent Peran | Kategori Skill Wajib | Skill Terpilih (Exact Name) | Lokasi Fisik Terpasang | Alasan Pemilihan & Cakupan Peran |
| :--- | :--- | :--- | :--- | :--- |
| **Orchestrator** | Superpowers planning, architecture, code review | `using-superpowers`<br>`writing-plans`<br>`requesting-code-review`<br>`ponytail` | `/home/ahmadbadawi/.gemini/config/skills/using-superpowers/SKILL.md`<br>`/home/ahmadbadawi/.gemini/config/skills/writing-plans/SKILL.md`<br>`/home/ahmadbadawi/.gemini/config/skills/requesting-code-review/SKILL.md`<br>`/home/ahmadbadawi/.gemini/config/skills/ponytail/SKILL.md` | Mengorkestrasi tahapan rekayasa, memastikan rencana terukur, mencegah *bloatware/over-engineering*, dan memimpin *peer-review*. |
| **Android Agent** | Kotlin, Android architecture, Compose, UI Tokens | `bauhaus-design`<br>`frontend-design`<br>`context7-auto-documentation` | `/home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/.agents/skills/bauhaus-design/SKILL.md`<br>`/home/ahmadbadawi/.gemini/config/skills/frontend-design/SKILL.md`<br>`/home/ahmadbadawi/.gemini/config/skills/context7-auto-documentation/SKILL.md` | Membangun antarmuka Jetpack Compose Neo-Bauhaus (grid matematis, border 2dp, kontras tinggi, typography sans-serif murni), integrasi ViewModel & StateFlow. |
| **Backend Agent** | Supabase, PostgreSQL, RLS, offline-sync | `supabase`<br>`supabase-postgres-best-practices` | `/home/ahmadbadawi/.gemini/config/skills/supabase/SKILL.md`<br>`/home/ahmadbadawi/.gemini/config/skills/supabase-postgres-best-practices/SKILL.md` | Memastikan skema PostgreSQL aman, RLS policies multi-tenant benar (`TO authenticated`, `USING (select auth.uid()) = user_id`), dan sinkronisasi data efisien. |
| **AI / RAG Agent** | RAG retrieval, OCR, context engineering | `context7-auto-documentation`<br>`pytorch-deep-learning` | `/home/ahmadbadawi/.gemini/config/skills/context7-auto-documentation/SKILL.md`<br>`/home/ahmadbadawi/.gemini/config/skills/pytorch-deep-learning/SKILL.md` | Mengatur retrieval 2-tier (FTS5 SQLite), isolasi prompt, proteksi formula matematika KaTeX/LaTeX, dan abstraksi model LiteRT/OCR. |
| **Web CMS Agent** | Next.js, frontend design, TypeScript, RBAC | `bauhaus-design`<br>`frontend-design`<br>`supabase` | `/home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/.agents/skills/bauhaus-design/SKILL.md`<br>`/home/ahmadbadawi/.gemini/config/skills/frontend-design/SKILL.md`<br>`/home/ahmadbadawi/.gemini/config/skills/supabase/SKILL.md` | Mengembangkan Web Admin CMS berbasis Next.js 15 App Router, proteksi RBAC JWT claims, dan tampilan Neo-Bauhaus yang konsisten. |
| **QA Agent** | TDD, debugging, verification | `test-driven-development`<br>`systematic-debugging`<br>`verification-before-completion`<br>`webapp-testing` | `/home/ahmadbadawi/.gemini/config/skills/test-driven-development/SKILL.md`<br>`/home/ahmadbadawi/.gemini/config/skills/systematic-debugging/SKILL.md`<br>`/home/ahmadbadawi/.gemini/config/skills/verification-before-completion/SKILL.md`<br>`/home/ahmadbadawi/.gemini/config/skills/webapp-testing/SKILL.md` | Menjalankan siklus TDD (Red-Green-Refactor), investigasi akar masalah, verifikasi build aktual sebelum klaim selesai, dan otomasi web testing. |
| **Security Agent** | Threat modeling, app security, database security | `owasp-security`<br>`supabase-postgres-best-practices` | `/home/ahmadbadawi/.gemini/config/skills/owasp-security/SKILL.md`<br>`/home/ahmadbadawi/.gemini/config/skills/supabase-postgres-best-practices/SKILL.md` | Audit kepatuhan OWASP MASVS & Top 10, sanitasi prompt injection, proteksi anti-surveillance gate data personal mahasiswa. |

---

## 2. Log Riwayat Aplikasi Skill pada Task Nyata (Execution Log)

### Task Record: #P0-P4-STABILIZE (Audit & Verifikasi Toolchain & Stabilisasi)
- **Task ID**: `TASK-AUDIT-001`
- **Agen Terlibat**: Orchestrator, Android, QA, Security, Web CMS.
- **Skill Diterapkan**:
  - `using-superpowers` (Orchestrator): Menetapkan disiplin verifikasi sebelum klaim.
  - `verification-before-completion` (QA): Menolak Python replica sebagai pengganti JVM test; menjalankan `./gradlew test` (59 tests pass) dan `./gradlew assembleDebug` (APK 90 MB).
  - `owasp-security` (Security): Memvalidasi Anti-Surveillance Gate dan isolasi RLS multi-tenant di Supabase & Next.js middleware.
  - `bauhaus-design` (Web CMS): Memperbaiki label transparan fixture metrik pada dashboard admin Next.js 15.
- **Bukti Nyata**:
  - Terminal `./gradlew test` -> 59 unit tests lulus dalam 1m 13s.
  - Terminal `npm run build` di `web-admin/` -> 6 static pages ter-prerender sukses.
- **Limitasi**: Supabase remote cloud project belum di-link (berstatus `BLOCKED`).

---

### Task Record: #P1-UI-COMPOSE (Implementasi Design System & UI Jetpack Compose Neo-Bauhaus)
- **Task ID**: `TASK-ANDROID-UI-001`
- **Agen Terlibat**: Android, Orchestrator, QA.
- **Skill Terpilih**:
  - `bauhaus-design` (`.agents/skills/bauhaus-design/SKILL.md`): Prinsip *form follows function*, grid matematis 8dp, border hitam 2dp, aksen warna Bauhaus primer (merah, kuning, biru), tipografi sans-serif bersih, anti-dekorasi berlebihan.
  - `frontend-design` (`.gemini/config/skills/frontend-design/SKILL.md`): Hirarki visual, kontras tinggi, state feedback interaktif, dan layout modular.
  - `test-driven-development` & `verification-before-completion` (QA): Validasi via pengujian unit logika UI State dan kompilasi APK debug.
- **Objektif**:
  1. Membangun fondasi Design System Jetpack Compose Neo-Bauhaus di `app/src/main/java/com/kerjaiaja/app/ui/theme/` (`Color.kt`, `Typography.kt`, `Shape.kt`, `Theme.kt`).
  2. Membangun reusable components: `BauhausButton`, `BauhausCard`, `PriorityBadge`, `BauhausSectionHeader`.
  3. Membangun `TaskListScreen` dengan visualisasi prioritas deterministik 4-faktor dan modal tambah tugas.
  4. Membangun `PhotoSolverScreen` untuk pratinjau OCR dan visualisasi 7 bagian solusi akademik (ADR-031).
  5. Menghubungkan navigasi `NeoBauhausBottomBar` di `MainActivity.kt`.
  6. Menguji logika filtering dan sorting UI di `TaskUiModelTest.kt`.
- **Hasil & Bukti Nyata**:
  - `app/src/test/java/com/kerjaiaja/app/ui/TaskUiModelTest.kt` dibuat dan lulus 2/2 tes.
  - Eksekusi `./gradlew test` -> **BUILD SUCCESSFUL (61/61 TESTS PASSED 100%)**.
  - Eksekusi `./gradlew assembleDebug` -> **BUILD SUCCESSFUL (APK 91 MB dibuat di `app/build/outputs/apk/debug/app-debug.apk`)**.
- **Limitasi**: Pratinjau kamera pada `PhotoSolverScreen` masih menggunakan simulasi citra tekstual dan belum terhubung ke preview hardware sensor CameraX fisik.

---

### Task Record: #P0-P4-GAP-CLOSURE (Sesi 3 - Penutupan Celah Rebaseline & Verifikasi Nyata)
- **Task ID**: `TASK-GAP-CLOSURE-003`
- **Agen Terlibat**: Orchestrator, Android Data & Sync, Supabase & Security, Build & Quality, Documentation.
- **Skill Terpilih & Lokasi Fisik**:
  - `using-superpowers` & `writing-plans` (`.gemini/config/skills/writing-plans/SKILL.md`)
  - `test-driven-development` (`.gemini/config/skills/test-driven-development/SKILL.md`)
  - `supabase` & `supabase-postgres-best-practices` (`.gemini/config/skills/supabase-postgres-best-practices/SKILL.md`)
  - `owasp-security` (`.gemini/config/skills/owasp-security/SKILL.md`)
  - `systematic-debugging` (`.gemini/config/skills/systematic-debugging/SKILL.md`)
  - `verification-before-completion` (`.gemini/config/skills/verification-before-completion/SKILL.md`)
  - `ponytail` (`.gemini/config/skills/ponytail/SKILL.md`)
- **Objektif & Hasil Nyata**:
  1. **Android Data & Sync:**
     - Membuat `RoomPersistenceReopenTest.kt` di JVM Robolectric (terbukti data persist setelah db ditutup dan dibuka kembali).
     - Mengimplementasikan `SupabaseTaskRemoteDataSource.kt` via PostgREST SDK resmi.
     - Menjadwalkan sync via `SyncScheduler.kt` dan menguji di `SyncSchedulerTest.kt` (3/3 pass).
     - Total unit test meningkat menjadi **74 tests across 16 classes (100% PASS)**.
  2. **Supabase & Keamanan:**
     - Migrasi skema kolaborasi tim `20261120000002_create_teams_and_collaboration_schema.sql` (SECURITY DEFINER helper functions mencegah infinite recursion RLS).
     - Menjalankan PostgreSQL 16 di Docker lokal: `rls_isolation_test.sql` (4/4 PASS), `team_rls_isolation_test.sql` (4/4 PASS).
     - Pemindaian secret menyeluruh: 0 temuan hardcoded secret.
  3. **Ukuran & Kualitas:**
     - Mengaktifkan ABI filtering `arm64-v8a`: APK release turun dari 67 MB ke **27 MB**; AAB release turun ke **17 MB** (anggaran $\le 25$ MB terpenuhi).
     - Investigasi lint: membuktikan crash 2 disabled checks merupakan bug resmi AGP 8.7.2 / Kotlin 2.0 (IssueTracker #343606775 & #342938810).
  4. **Tata Kelola Lingkup:**
     - Foto Soal/OCR, Vision Lab, dan Web Admin CMS ditandai tegas sebagai **`PASCA-MVP / DITAHAN MENUNGGU KEPUTUSAN PENGGUNA`**.
     - Anotasi ADR-001 s/d ADR-004 pra-rebaseline dikonsolidasikan ke kanonikal ADR.
- **Bukti Terminal Nyata**:
  - `./gradlew test` -> **74/74 unit tests PASSED**.
  - `docker run postgres:16-alpine` -> RLS isolation SQL test 8/8 PASSED.
  - `./gradlew assembleRelease` -> APK 27 MB.
  - `./gradlew bundleRelease` -> AAB 17 MB.
  - `./gradlew lintDebug` -> 0 errors, 67 warnings.


