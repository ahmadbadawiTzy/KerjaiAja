# Rencana Fase Implementasi (Implementation Phases) — KerjaiAja

Dokumen ini menguraikan tahapan aksi eksekusi teknis untuk merealisasikan aplikasi **KerjaiAja**.

---

## Tahap 0: Fondasi Workspace & Desain Arsitektur (Status: Sedang Berlangsung)
- **Tujuan:** Menyiapkan struktur direktori yang terisolasi, merumuskan spesifikasi produk, dokumen arsitektur Clean Architecture, strategi Adaptive Local AI, dan konvensi rekayasa perangkat lunak.
- **Kondisi:** Repositori siap tanpa adanya kode sumber aplikasi atau dependensi build yang terpasang prematur.

---

## Tahap 1: Inisialisasi Proyek Android & Modul Core (Estimasi Sprint: 1-2)
- **Aktivitas:**
  1. Bootstrapping proyek Android Studio dengan Gradle Kotlin DSL (`build.gradle.kts`) dan Version Catalog (`gradle/libs.versions.toml`).
  2. Setup arsitektur multi-layer (presentation, domain, data) dan dependensi Hilt.
  3. Konfigurasi Material 3 Theme (`Color.kt`, `Theme.kt`, `Type.kt`).
  4. Pembuatan entitas Room `SubjectEntity` dan `TaskEntity` beserta DAO dan unit test in-memory.
  5. Pembangunan UI Compose untuk dasbor tugas dan layar tambah mata kuliah.
- **Kriteria Selesai (DoD):** Pengguna dapat membuat mata kuliah dan tugas akademik baru secara offline dengan persistensi Room yang teruji.

---

## Tahap 2: Dekonstruksi Manual, Jadwal Belajar & Checklist (Estimasi Sprint: 3)
- **Aktivitas:**
  1. Implementasi entitas `SubtaskEntity` dan logika perhitungan progres persentase tugas.
  2. Pembuatan komponen antarmuka *Breakdown Checklist* interaktif.
  3. Implementasi *Focus Timer* lokal yang dapat menghitung durasi kerja per tugas.
  4. Pembuatan modul *Submission Checklist* untuk memvalidasi kelengkapan berkas sebelum pengumpulan.
- **Kriteria Selesai (DoD):** Tugas dapat dipecah menjadi sub-langkah konkret secara manual dan diverifikasi kelengkapannya dengan checklist terpadu.

---

## Tahap 3: Integrasi Adaptive Local AI Engine & Local RAG (Estimasi Sprint: 4-5)
- **Aktivitas:**
  1. Pembuatan kontrak `AiEngine` di modul domain dan implementasi `FakeAiEngine` untuk pengujian otomatis.
  2. Implementasi `DeviceCapabilityDetector` untuk memverifikasi kapasitas RAM dan kompatibilitas API 29+.
  3. Pembuatan `ModelDownloadWorker` berbasis WorkManager dengan verifikasi hash integritas SHA-256.
  4. Integrasi tabel virtual SQLite FTS di Room untuk pencarian materi/catatan kuliah lokal.
  5. Integrasi runtime native `LiteRT-LM` dan model terkuantisasi `Gemma 3 1B IT`.
  6. Pembangunan Prompt Builder terstruktur dengan parsing keluaran skema JSON.
  7. Penanganan skenario *graceful fallback* jika perangkat tidak memenuhi syarat.
- **Kriteria Selesai (DoD):** Model AI lokal dapat diunduh on-demand, divalidasi hashnya, dan berhasil memberikan saran sub-tugas terstruktur dalam format JSON di perangkat uji fisik tanpa kebocoran memori.

---

## Tahap 4: Proyek Kelompok & Sinkronisasi Cloud Opsional (Estimasi Sprint: 6)
- **Aktivitas:**
  1. Penambahan field delegasi anggota tim pada sub-tugas dan tampilan matriks tanggung jawab kelompok.
  2. Inisialisasi SDK klien Supabase untuk otentikasi opsional dan pencadangan data ke cloud PostgreSQL.
  3. Pengujian ketahanan sinkronisasi offline-ke-online dengan resolusi konflik data.
- **Kriteria Selesai (DoD):** Proyek kelompok dapat ditugaskan ke anggota tim secara terstruktur, dan pengguna memiliki opsi mencadangkan data ke Supabase jika diinginkan.

---

## Tahap 5: Pengujian Menyeluruh, Optimasi Performa & Rilis (Estimasi Sprint: 7)
- **Aktivitas:**
  1. Eksekusi pengujian otomatis menyeluruh (Unit Test, Compose Widget Test, Integration Test).
  2. Profiling konsumsi memori dan daya baterai selama inferensi AI di perangkat RAM 4 GB fisik.
  3. Audit keamanan kode sesuai standar OWASP Mobile ASVS.
  4. Penyusunan berkas rilis Android App Bundle (AAB), Proguard/R8 shrinking rules, dan kesiapan publikasi.
- **Kriteria Selesai (DoD):** Aplikasi stabil, bebas dari memory leak atau crash, lulus seluruh suite pengujian, dan siap untuk deployment.
