# Spesifikasi Kebutuhan Produk (Product Requirements) — KerjaiAja

Dokumen ini mendefinisikan kebutuhan fungsional dan non-fungsional aplikasi **KerjaiAja**.

---

## 1. Kebutuhan Fungsional (Functional Requirements)

### FR-01: Manajemen Tugas Akademik (Academic Task Management)
- Sistem wajib menyediakan kemampuan membuat, membaca, memperbarui, dan menghapus (*CRUD*) tugas akademik.
- Setiap tugas wajib memiliki metadata:
  - Judul tugas (*title*)
  - Deskripsi/instruksi tugas (*description/prompt*)
  - Mata kuliah pengampu (*subject reference*)
  - Batas waktu pengumpulan (*due date & time*)
  - Skala prioritas (Rendah, Sedang, Tinggi, Mendesak)
  - Status pengerjaan (Belum Dimulai, Sedang Dikerjakan, Siap Review, Selesai)
  - Estimasi durasi pengerjaan (dalam menit/jam)
  - Tipe tugas (Individu / Kelompok / Tugas Mandiri / Proyek Akhir)

### FR-02: Manajemen Mata Kuliah / Pelajaran (Subject Management)
- Pengelompokan tugas berdasarkan mata kuliah atau pelajaran.
- Informasi mata kuliah mencakup: nama mata kuliah, kode, nama pengajar/dosen, jadwal pertemuan mingguan, ruang kelas/link telekonferensi, dan skema bobot penilaian.
- Visualisasi warna penanda (*color tagging*) untuk diferensiasi visual cepat di antarmuka.

### FR-03: Dekonstruksi Tugas (Assignment Breakdown)
- Kemampuan memecah tugas besar menjadi sub-tugas berjenjang (*actionable subtasks*).
- Setiap sub-tugas memiliki tenggat waktu parsial (*intermediate milestone*) dan indikator ketercapaian.
- Visualisasi progres penyelesaian tugas berbasis persentase penyelesaian sub-tugas.

### FR-04: Perencanaan Belajar (Study Planning)
- Pemetaan alokasi waktu pengerjaan tugas ke dalam jadwal harian dan mingguan.
- Fitur penghitung mundur sesi fokus (*focus session timer / Pomodoro-friendly*) yang terikat langsung ke tugas yang sedang aktif.
- Peringatan dini (*smart reminder*) sebelum tenggat waktu terlewati.

### FR-05: Asisten Belajar AI Lokal (AI Learning Assistant)
- Antarmuka interaksi asisten berbasis teks yang berjalan lokal di perangkat (*on-device*).
- Kemampuan menyarankan pemecahan tugas (*automatic breakdown recommendation*) berdasarkan teks instruksi tugas yang dimasukkan.
- Kemampuan merangkum catatan materi lokal untuk mempercepat pemahaman konteks pengerjaan.
- *Strict safety:* AI dirancang memberikan panduan struktural (*scaffolding*), bukan menghasilkan plagiasi dokumen secara otomatis.

### FR-06: Manajemen Proyek Kelompok (Group Project Management)
- Pemisahan tugas kelompok dengan matriks pembagian tanggung jawab (*responsibility matrix*).
- Pencatatan nama anggota tim dan penugasan spesifik pada masing-masing sub-tugas.
- Pelacakan status pengerjaan setiap anggota untuk memitigasi risiko keterlambatan kolektif.

### FR-07: Daftar Periksa Pengumpulan (Submission Checklist)
- Verifikasi pra-pengumpulan tugas (*pre-flight submission checklist*):
  - Validasi format file yang diminta (misal: PDF, ZIP, DOCX, link repositori).
  - Pengecekan konvensi penamaan file (misal: `NIM_Nama_Tugas1.pdf`).
  - Verifikasi kelengkapan rubrik (halaman judul, daftar pustaka, lampiran data).
  - Pencatatan bukti pengumpulan (tanggal kirim & catatan konfirmasi portal akademik).

---

## 2. Kebutuhan Non-Fungsional (Non-Functional Requirements)

### NFR-01: Arsitektur & Offline-First
- Aplikasi harus berfungsi penuh secara *offline* tanpa memerlukan koneksi jaringan internet untuk seluruh fitur manajemen data lokal.
- Semua data tersimpan aman di basis data lokal SQLite (Room).

### NFR-02: Batasan Kinerja & Sumber Daya Perangkat
- Waktu peluncuran awal aplikasi (*cold start*) < 1.5 detik pada perangkat uji kelas menengah (mid-range).
- Tingkat *frame rate* antarmuka stabil di 60/120 FPS tanpa terjadi *jank* selama transisi Compose.
- Konsumsi RAM saat aplikasi aktif normal (tanpa AI) < 150 MB.
- Saat modul inferensi Local AI aktif, penggunaan memori tambahan tidak boleh memicu `OutOfMemoryError` (alokasi memori inferensi dibatasi ketat sesuai profil perangkat).

### NFR-03: Kompatibilitas Sistem Operasi
- Minimum Android SDK: **Android 10 (API level 29)**.
- Target Android SDK: Android 14+ (API level 34+).
- Kompatibel dengan arsitektur CPU 64-bit (`arm64-v8a`).

### NFR-04: Privasi & Keamanan Data
- Data pribadi, teks tugas, dan histori pencarian tidak dikirim ke pihak ketiga secara tersembunyi.
- Dukungan backup/sinkronisasi cloud bersifat opt-in (pilihan sadar pengguna) melalui backend Supabase terenkripsi.
