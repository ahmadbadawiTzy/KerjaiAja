# Catatan Keputusan Arsitektur (Architecture Decision Records) — KerjaiAja

Dokumen ini mencatat keputusan-keputusan arsitektural penting (*ADRs*) yang telah disepakati untuk proyek **KerjaiAja**.

---

## ADR-001: Pemilihan Platform Android Native, Kotlin, dan Jetpack Compose

- **Status:** DITERIMA
- **Konteks:** Proyek membutuhkan antarmuka yang sangat responsif, akses langsung ke akselerasi perangkat keras lokal (GPU/NPU) untuk eksekusi model on-device, serta adopsi sistem desain modern Material 3.
- **Keputusan:** Mengembangkan aplikasi sebagai **Android Native** murni menggunakan bahasa **Kotlin** dan toolkit deklaratif **Jetpack Compose** dengan **Material 3**.
- **Konsekuensi Positif:**
  - Integrasi tanpa hambatan dengan runtime machine learning lokal (C++ bindings via NDK/LiteRT).
  - Performa rendering grafis optimal dan konsumsi memori terprediksi.
  - Desain modern adaptif dengan *Dynamic Color* (Material You).
- **Konsekuensi Negatif:**
  - Aplikasi terbatas pada ekosistem Android (tidak lintas platform ke iOS/Web pada tahap ini).

---

## ADR-002: Penerapan Clean Architecture + Pola MVVM

- **Status:** DITERIMA
- **Konteks:** Aplikasi memiliki alur logika yang kompleks (manajemen tugas, perhitungan jadwal, alur RAG, dan isolasi AI runtime) yang harus mudah diuji secara independen tanpa ketergantungan pada Android UI lifecycle.
- **Keputusan:** Menggunakan **Clean Architecture** berlapis (Presentation, Domain, Data) dengan pola **MVVM (Model-View-ViewModel)** di presentation layer.
- **Konsekuensi Positif:**
  - Logika bisnis murni di Domain layer dapat diuji dengan Unit Test cepat (~70% cakupan).
  - Aliran data satu arah (*Unidirectional Data Flow*) mencegah bug desinkronisasi state UI.
  - Memudahkan penggantian implementasi runtime AI tanpa merusak modul UI.
- **Konsekuensi Negatif:**
  - Penambahan boilerplate kelas awal (UseCases, Repositories, Data Mappers).

---

## ADR-003: Basis Data Lokal Menggunakan Room Database dan SQLite FTS

- **Status:** DITERIMA
- **Konteks:** Aplikasi harus mengusung prinsip *Offline-First* dan membutuhkan kemampuan pencarian teks cepat pada materi kuliah untuk menyusun prompt RAG lokal.
- **Keputusan:** Menggunakan **Room Persistence Library** di atas SQLite lokal dengan aktivasi ekstensi tabel virtual **SQLite FTS (Full-Text Search)**.
- **Konsekuensi Positif:**
  - Akses data relasional aman terhadap tipe data (*type-safe*).
  - Pencarian materi lokal berkecepatan sub-milidetik untuk injeksi konteks AI.
  - Zero latency jaringan untuk seluruh alur pencatatan tugas.
- **Konsekuensi Negatif:**
  - Diperlukan pengelolaan migrasi skema basis data Room yang disiplin saat rilis pembaruan.

---

## ADR-004: Adaptive On-Device Local AI Engine (Gemma 3 1B IT + LiteRT-LM)

- **Status:** DITERIMA
- **Konteks:** Privasi data akademik pelajar sangat sensitif, dan biaya operasional API cloud berkelanjutan tidak diinginkan. Aplikasi membutuhkan model bahasa yang dapat memecah tugas akademik secara terstruktur langsung di ponsel.
- **Keputusan:** Menggunakan model **Gemma 3 1B IT** yang dijalankan melalui runtime **LiteRT-LM** (Google AI Edge) secara lokal di perangkat dengan arsitektur adaptif (RAM target 4–6 GB, API 29+). Aplikasi inti tetap 100% fungsional tanpa AI jika perangkat tidak mendukung (*graceful fallback*).
- **Konsekuensi Positif:**
  - Bebas biaya langganan cloud AI API bagi pengguna maupun pengembang.
  - Privasi data 100% terjaga di perangkat pengguna.
  - Kemampuan inferensi tetap berfungsi saat tidak ada sinyal internet.
- **Konsekuensi Negatif:**
  - Keterbatasan kapasitas penalaran model 1B parameter dibandingkan model cloud raksasa (mitigasi: rekayasa prompt ketat & RAG lokal).
  - Perlu perhatian ekstra terhadap konsumsi memori dan suhu perangkat.

---

## ADR-005: Pengunduhan Model On-Demand dan Verifikasi Integritas SHA-256

- **Status:** DITERIMA
- **Konteks:** Membundel bobot model biner (~600–900 MB) ke dalam berkas APK instalasi awal akan membuat aplikasi terlalu besar dan tidak ramah kuota.
- **Keputusan:** Mendistribusikan model biner secara **on-demand** terpisah setelah aplikasi terpasang, dengan validasi integritas checksum **SHA-256** wajib sebelum model diaktifkan.
- **Konsekuensi Positif:**
  - Ukuran APK awal sangat ramping (< 25 MB).
  - Mencegah runtime crash akibat berkas model korup atau terpotong saat pengunduhan.
- **Konsekuensi Negatif:**
  - Pengguna harus mengunduh data tambahan saat pertama kali mengaktifkan fitur AI.

---

## ADR-006: Supabase sebagai Backend Cloud Sinkronisasi Opsional

- **Status:** DITERIMA
- **Konteks:** Pengguna mungkin menginginkan pencadangan data lintas perangkat atau kerja kelompok kolaboratif di masa mendatang, namun sistem tidak boleh bergantung secara wajib pada server.
- **Keputusan:** Menetapkan **Supabase** (PostgreSQL, Auth, Storage) sebagai backend cloud sekunder yang bersifat **opsional (opt-in)**.
- **Konsekuensi Positif:**
  - Arsitektur backend cloud open-source yang hemat biaya, aman dengan Row-Level Security (RLS).
  - Pemisahan yang bersih antara data lokal offline dan replikasi cloud.
- **Konsekuensi Negatif:**
  - Menuntut penanganan logika sinkronisasi dan resolusi konflik data saat mode cloud diaktifkan.
