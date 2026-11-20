# Distribusi & Manajemen Model AI (Model Distribution) — KerjaiAja [SUPERSEDED]

> [!WARNING]
> **STATUS ARSITEKTUR: SUPERSEDED (DIGANTIKAN)**  
> Sesuai [ADR-004: Backend AI Inference Architecture](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-004-BACKEND-AI-INFERENCE.md), mekanisme pengunduhan model biner on-device ke ponsel pengguna tidak lagi digunakan. Komputasi AI dijalankan secara terpusat di Go Backend, sehingga klien Android tidak perlu lagi mengunduh atau menyimpan bobot model biner. Dokumen ini dipertahankan sebagai catatan audit historis.

---

## 1. Strategi Pengunduhan On-Demand (On-Demand Download)

### 1.1 Rasional Ukuran Berkas APK
- Membundel (*bundling*) berkas model biner (600–900 MB) ke dalam APK awal akan menyebabkan:
  - Ukuran unduhan awal di Google Play Store terlalu besar dan memicu keengganan pengguna untuk memasang aplikasi.
  - Membebani pengguna yang perangkatnya tidak memenuhi syarat perangkat keras untuk menjalankan AI.
  - Kesulitan melakukan rilis pembaruan perbaikan bug (*hotfix*) aplikasi tanpa memaksa pengguna mengunduh ulang model biner.
- **Keputusan:** APK awal hanya berukuran < 25 MB (hanya berisi kode Kotlin, UI Compose, dan pustaka runtime). Bobot model AI diunduh secara terpisah melalui mekanisme on-demand setelah aplikasi terpasang dan pengguna memberikan persetujuan sadar.

---

## 2. Verifikasi Integritas Model (SHA-256 Checksum)

Integritas berkas model biner sangat kritikal untuk mencegah kegagalan inisialisasi runtime C++ atau potensi eksekusi berkas korup:

1. **Hash Manifest Terverifikasi:**  
   Aplikasi menyimpan nilai checksum resmi hash **SHA-256** untuk setiap versi bobot model yang didukung.
2. **Proses Verifikasi Mandiri:**  
   Setelah unduhan berkas selesai, background worker secara otomatis menghitung *hash digest* SHA-256 dari berkas lokal yang tersimpan.
3. **Penyaringan Keamanan:**
   - Jika hash cocok persis: Berkas ditandai `VALIDATED` dan dipindahkan ke direktori eksekusi model aktif.
   - Jika hash tidak cocok (korup akibat gangguan koneksi atau berkas tidak lengkap): Berkas lokal langsung dihapus secara aman, status diubah ke `DOWNLOAD_FAILED_CHECKSUM`, dan pengguna diberi opsi untuk mengunduh ulang.

---

## 3. Manajemen Siklus Hidup & Penyimpanan Berkas (Model Storage Lifecycle)

### 3.1 Lokasi Penyimpanan Aman
- Berkas model biner disimpan di direktori privat internal aplikasi:
  `context.filesDir.resolve("models/gemma3-1b-it.litertlm")`
- Direktori ini bersifat terisolasi (*sandboxed*), tidak dapat diakses atau diubah oleh aplikasi pihak ketiga tanpa hak akses root.
- **Pengecekan Ruang Penyimpanan:** Sebelum unduhan dimulai, `DeviceCapabilityDetector` wajib memvalidasi ketersediaan ruang penyimpanan internal bebas minimal **2.5 GB** ([ADR-001](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-001-LOCAL-AI-CAPABILITY-DETECTION.md)) untuk menampung berkas unduhan dan ruang buffer runtime.

### 3.2 Ketahanan Pengunduhan via WorkManager
- Pengunduhan dijalankan sebagai `CoroutineWorker` di dalam **Android WorkManager**.
- Mendukung fitur:
  - Batasan jaringan: Hanya mengunduh via Wi-Fi secara default (dapat disesuaikan oleh pengguna).
  - Melanjutkan unduhan terputus (*resumable HTTP range requests*).
  - Notifikasi progres transparan di system tray Android.

### 3.3 Kontrol Pengguna & Manajemen Cache
- Pengguna diberikan kontrol penuh di menu Pengaturan:
  - Melihat ukuran ruang penyimpanan yang digunakan oleh berkas model.
  - Opsi menghapus berkas model kapan saja untuk menghemat ruang memori internal.
  - Opsi memeriksa versi model terbaru dan mengunduh ulang jika diperlukan.
