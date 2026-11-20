# ADR-019: Tata Kelola & Mekanisme Unduhan Bobot Model On-Demand (On-Demand Model Download Management)

## Status
**ACCEPTED** (2026-11-20) — Menetapkan arsitektur distribusi bobot model AI on-device.

---

## 1. Konteks (Context)
Model kecerdasan buatan on-device seperti Google Gemma 4 E2B atau Gemma 3 1B memiliki ukuran berkas terkuantisasi antara ~1.1 GB hingga ~1.6 GB. Membundel bobot model secara langsung ke dalam file APK/AAB akan:
1. Menyebabkan ukuran download awal di Google Play Store membengkak drastis, memicu penolakan unduhan oleh mahasiswa dengan kuota internet terbatas.
2. Membebani ponsel mahasiswa yang memiliki kapasitas penyimpanan internal terbatas.
3. Melanggar prinsip *free choice* di mana pengguna berhak memilih untuk menggunakan fitur AI atau hanya menggunakan engine produktivitas non-AI.

---

## 2. Keputusan (Decision)
KerjaiAja menetapkan **Arsitektur Pemisahan Bobot Model (Decoupled Weight Management)**:

1. **Model DILARANG Dititipkan di APK:**  
   APK/AAB dirilis dalam keadaan bersih tanpa file model (`.litertlm` atau bobot biner), dengan target ukuran instalasi < 20 MB.
2. **Persetujuan Eksplisit Pengguna (Explicit User Consent):**  
   Aplikasi **TIDAK MENGUNDUH MODEL SECARA OTOMATIS** saat pertama kali dibuka. Unduhan hanya dapat dipicu secara sengaja oleh pengguna melalui menu Pengaturan (*Settings -> Model AI On-Device*).
3. **Pemeriksaan Pra-Unduhan (Pre-Flight Checks):**  
   Sebelum proses unduhan dimulai, sistem memvalidasi:
   - Ruang penyimpanan internal bebas minimal 3.0 GB.
   - Status baterai $\ge 30\%$ atau sedang terhubung ke pengisi daya.
   - Opsi preferensi "Unduh hanya melalui Wi-Fi" aktif secara default.
4. **Penyimpanan Terisolasi Privat (App-Specific Storage):**  
   Berkas model diunduh ke direktori privat aplikasi (`context.filesDir/models/`) agar tidak dapat diakses oleh aplikasi pihak ketiga lain di perangkat.
5. **Validasi Integritas Kriptografis (Checksum Verification):**  
   Setelah unduhan selesai, sistem melakukan validasi checksum SHA-256 terhadap metadata resmi sebelum menandai model sebagai `READY`.
6. **Siklus Status Model Eksplisit (Model Lifecycle State):**  
   - `NOT_DOWNLOADED`
   - `DOWNLOADING(progress: Int)` (Mendukung *resumable download*)
   - `VERIFYING_CHECKSUM`
   - `READY`
   - `INCOMPATIBLE` (Jika RAM fisik tidak memadai)
   - `ERROR(reason: String)`
7. **Hak Penghapusan Instan (Instant Deletion):**  
   Pengguna dapat menghapus berkas bobot model kapan saja dengan satu ketukan tombol di menu Pengaturan untuk membebaskan ruang disk ponsel tanpa menghapus data tugas.

---

## 3. Konsekuensi (Consequences)
### Positif:
- Distribusi Play Store sangat cepat dan hemat kuota awal.
- Menghormati privasi dan kebebasan pengguna atas ruang penyimpanan ponsel mereka.
- Keamanan integritas model terjamin dari korupsi data unduhan.
### Negatif:
- Pengguna yang ingin menggunakan fitur AI membutuhkan waktu tambahan untuk mengunduh berkas model melalui koneksi internet stabil sekali di awal.
