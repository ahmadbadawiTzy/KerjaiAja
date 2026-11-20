# ADR-023: Dual Experience Product Architecture (Full Productivity Without AI & Contextual On-Device Assistant)

## Status
Accepted

## Konteks
Sebagian besar aplikasi "AI" modern memaksa pengguna mengandalkan fitur AI untuk fungsi-fungsi dasar aplikasi, atau bahkan memblokir pengalaman pengguna jika perangkat tidak mampu menjalankan AI. Di sisi lain, pelajar dan mahasiswa memiliki beragam tingkatan perangkat Android (dari perangkat budget dengan RAM terbatas hingga flagship). KerjaiAja harus adil, inklusif, dan memberikan nilai produktivitas maksimal bagi seluruh pengguna.

## Keputusan
Kami menetapkan arsitektur produk **Dual Experience**:

### 1. Mode A: Produktivitas Penuh Tanpa AI (Full Productivity Without AI)
- Menjadi fondasi default aplikasi.
- Seluruh fitur inti produktivitas berfungsi 100% tanpa perlu model AI atau koneksi ke engine inferensi:
  - Manajemen tugas, sub-tugas, checklist, dan tenggat waktu.
  - Kalender akademik dan visualisasi jadwal kuliah/sekolah.
  - Pengorganisasian mata pelajaran, tag, dan estimasi waktu.
  - **Deterministic Priority Engine:** Perhitungan prioritas tugas berbasis bobot matematis yang terbukti, transparan, dan dapat dijelaskan (deadline, bobot nilai, kesulitan, dependensi).
  - Sinkronisasi cloud Supabase (jika login) dan kolaborasi tim.
- Ukuran download awal aplikasi tetap sangat ramping (~15–25 MB).

### 2. Mode B: Produktivitas dengan Asisten Kerja On-Device Gemma (Contextual Assistant)
- Bersifat **opsional (opt-in)** dan hanya dapat diaktifkan jika perangkat memenuhi syarat hardware.
- Menambahkan kapabilitas analisis konteks tugas lokal:
  - *Ask About My Work:* Tanya status beban kerja dan progres tugas saat ini.
  - *Task Breakdown:* Pemecahan otomatis tugas besar menjadi sub-tugas terstruktur.
  - *Smart Planning:* Rekomendasi blok waktu belajar yang optimal.
  - *Contextual Recommendations:* Saran tugas berikutnya berdasarkan tenggat dan tingkat energi pengguna.
  - *Work Review:* Pengecekan kelengkapan checklist tugas sebelum pengumpulan.
- Menggunakan model Google Gemma (Gemma 3 1B IT / 270M) yang diunduh terpisah secara on-demand.

## Konsekuensi
### Positif
- Tidak ada pengguna yang terdiskriminasi karena spesifikasi hardware yang rendah.
- Aplikasi memiliki nilai jual yang kokoh sebagai tool produktivitas murni, bukan sekadar "gimmick AI".
- Performa UI tetap gesit karena logika deterministik menangani operasi harian tanpa beban komputasi AI yang berat.

### Negatif / Mitigasi
- Kode antarmuka (UI) harus menangani dua status (AI aktif vs AI non-aktif) secara elegan (dimitigasi dengan state pattern dan UI placeholder yang adaptif).
