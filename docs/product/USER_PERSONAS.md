# Profil Pengguna (User Personas) — KerjaiAja
## Persona Pelajar & Mahasiswa: Kebutuhan Produktivitas & Pendamping Belajar AI

Dokumen ini mendefinisikan profil persona representatif untuk platform **KerjaiAja**, mencakup skenario penggunaan Mode A (Produktivitas Tanpa AI) dan Mode B (Produktivitas dengan AI On-Device & Foto Soal).

---

## 1. Persona 1: Mahasiswa Teknik Informatika (Power User — Mode B)
- **Nama:** Arya Pratama (21 tahun)
- **Status:** Mahasiswa Semester 5 Teknik Informatika, Universitas Negeri di Bandung
- **Perangkat:** Smartphone Android Mid-to-High Tier (RAM 8 GB, Snapdragon 778G, Storage 128 GB)
- **Karakteristik & Tantangan:**
  - Menghadapi 6 mata kuliah praktikum dengan tugas coding, laporan mingguan, dan kuis kalkulus/probabilitas.
  - Sering mengalami *procrastination paralysis* saat melihat instruksi tugas akhir modul yang sangat panjang.
  - Membutuhkan pemecahan langkah pengerjaan (*task breakdown*) dan terkadang bingung memahami soal kalkulus matematika diskrit bertingkat.
- **Bagaimana KerjaiAja Membantu:**
  - Mengunduh Google Gemma 3 1B IT on-device.
  - Memanfaatkan **Foto Soal & Pembahasan** untuk memotret soal logika matematika diskrit: sistem memberikan pembahasan langkah demi langkah dan verifikasi konsep.
  - Menggunakan **Asisten Kerja** (*Task Breakdown*) untuk mengonversi tugas besar pengembangan web menjadi sub-tugas terukur.
  - Sinkronisasi tugas kelompok praktikum dengan teman tim via Supabase Realtime.

---

## 2. Persona 2: Siswi SMA Kelas 12 (Budget Device — Mode A & Jalur OCR)
- **Nama:** Nabila Putri (17 tahun)
- **Status:** Siswi Kelas 12 SMA Negeri di Solo, sedang persiapan SNBT/UTBK
- **Perangkat:** Smartphone Android Entry-Level / Budget (RAM 4 GB, Helio G85, Storage 64 GB)
- **Karakteristik & Tantangan:**
  - Memiliki kuota internet terbatas dan ruang memori ponsel terbatas.
  - Menghadapi ratusan soal latihan persiapan ujian (Fisika, Kimia, Biologi, Bahasa Indonesia).
  - Takut aplikasi produktivitas membuat ponselnya lambat atau baterai cepat habis.
- **Bagaimana KerjaiAja Membantu:**
  - Menikmati **Mode A (Produktivitas Penuh Tanpa AI)** secara default: mencatat jadwal tryout, to-do list belajar harian, dan timer fokus Pomodoro tanpa beban memori.
  - Kualifikasi `LIMITED`: Memilih mengunduh model ultra-ringan **Gemma 3 270M**.
  - Menggunakan **Foto Soal (Jalur A: OCR-First)** untuk memotret soal teks Biologi dan Bahasa Indonesia: teks diekstraksi via ML Kit on-device, dan pembahasan konseptual disajikan secara ringkas dan hemat baterai.

---

## 3. Persona 3: Mahasiswi Ilmu Komunikasi & Anggota Tim Proyek (Collaborative User — Mode A & B)
- **Nama:** Salsa Dwi (20 tahun)
- **Status:** Mahasiswi Semester 3 Ilmu Komunikasi, Universitas Swasta di Jakarta
- **Perangkat:** Smartphone Android Mid-Range (RAM 6 GB, Dimensity 7050, Storage 128 GB)
- **Karakteristik & Tantangan:**
  - Banyak tugas kelompok (pembuatan kampanye media, riset observasi lapangan).
  - Kerap frustrasi akibat *free-rider* (teman kelompok yang tidak aktif) dan komunikasi tugas kelompok yang tercecer di grup chat.
- **Bagaimana KerjaiAja Membantu:**
  - Menggunakan fitur **Team Workspace** berbasis Supabase Realtime & RLS untuk membagi deliverable kelompok secara transparan.
  - Menggunakan **Deterministic Priority Engine** untuk memantau tugas kelompok yang mendekati deadline secara objektif tanpa bias personal.
  - Menggunakan **Foto Soal** untuk memotret artikel/bacaan studi kasus Sosiologi/Komunikasi dan memahami konsep relasi sosial yang ditanyakan.
