# Roadmap Pengembangan (Development Roadmap) — KerjaiAja

Dokumen ini menjabarkan peta jalan pengembangan aplikasi **KerjaiAja** dari tahap inisialisasi hingga rilis produksi.

---

## 1. Ikhtisar Tahapan (Phases Overview)

```text
[Fase 0: Fondasi Workspace] ➡️ [Fase 1: Core Non-AI App] ➡️ [Fase 2: Breakdown & Planner] 
                                                                        ⬇️
[Fase 5: Release Readiness]  ⬅️ [Fase 4: Group & Submission] ⬅️ [Fase 3: Local AI Integration]
```

---

## 2. Rincian Fase Kerja

### Fase 0: Inisialisasi Workspace & Perancangan Arsitektur (Status: Aktif)
- **Target:** Membangun ruang kerja terisolasi, merumuskan spesifikasi produk, dokumen arsitektur Clean Architecture, strategi Local AI, dan konvensi proyek.
- **Deliverables:** Repositori terstruktur, berkas `.gitignore`, seluruh dokumen di `docs/` dan `planning/`.
- **Aturan Ketat:** Belum ada penulisan kode Kotlin atau skrip build Gradle.

### Fase 1: Bootstrap Project & Modul Core Manajemen Tugas
- **Target:** Inisialisasi proyek Android Studio dengan Gradle Version Catalog (`libs.versions.toml`), penyiapan Hilt, Room Database, dan modul UI dasar.
- **Deliverables:**
  - Konfigurasi Room Database (`subjects`, `tasks`).
  - UseCases dan Repository untuk Task & Subject CRUD.
  - Tampilan layar daftar tugas dan mata kuliah dengan Jetpack Compose + Material 3.

### Fase 2: Dekonstruksi Manual, Perencanaan Belajar & Checklist
- **Target:** Membangun alur kerja pemecahan tugas manual dan manajemen waktu fokus.
- **Deliverables:**
  - Entitas dan UI untuk `subtasks` dan `submission_checklists`.
  - Fitur pengatur waktu fokus (*Focus Timer*) terintegrasi dengan tugas aktif.
  - Perhitungan indikator progres tugas otomatis.

### Fase 3: Integrasi Adaptive Local AI Engine
- **Target:** Menghubungkan runtime `LiteRT-LM` dan model on-device `Gemma 3 1B IT`.
- **Deliverables:**
  - Abstraksi `AiEngine` dan implementasi `LiteRtLmEngine`.
  - Worker pengunduhan model on-demand dengan validasi hash SHA-256 via WorkManager.
  - Integrasi query konteks SQLite FTS untuk Local RAG.
  - Fitur rekomendasi dekonstruksi instruksi tugas otomatis.
  - Mekanisme graceful fallback untuk perangkat non-kompatibel.

### Fase 4: Manajemen Proyek Kelompok & Sinkronisasi Opsional
- **Target:** Fasilitas kerja kelompok dan integrasi backend opsional Supabase.
- **Deliverables:**
  - Matriks pembagian peran sub-tugas kelompok di basis data lokal.
  - Integrasi klien Supabase (opsional) untuk backup data terenkripsi.
  - Ekspor ringkasan progres tugas ke format teks/PDF ringkas.

### Fase 5: Pengujian Menyeluruh, Optimasi Performa & Rilis
- **Target:** Verifikasi stabilitas, profiling memori on-device, dan persiapan peluncuran.
- **Deliverables:**
  - Suite pengujian lengkap (Unit, Robolectric, Compose UI).
  - Profiling alokasi memori inferensi AI pada perangkat RAM 4 GB fisik.
  - Pembuatan signed Android App Bundle (AAB) dan kesiapan rilis Google Play.
