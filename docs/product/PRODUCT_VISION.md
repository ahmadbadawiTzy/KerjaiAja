# Visi Produk — KerjaiAja

> *"Turn academic assignments into clear, manageable, and collaborative actions without guilt."*

---

## 1. Latar Belakang & Pernyataan Masalah

Mahasiswa dan pelajar di era modern menghadapi beban akademik yang semakin kompleks. Tantangan belajar tidak hanya bersumber dari banyaknya tugas, melainkan dari multi-faktor yang saling bertumpuk:
1. **Beban Kognitif Berlebih (*Cognitive Overload*):** Instruksi tugas yang panjang, ambigu, dan tersebar di berbagai platform menimbulkan keraguan awal (*friction to start*), memicu penundaan pengerjaan (*procrastination*), serta membebani mental mahasiswa.
2. **Ketiadaan Prioritisasi yang Transparan:** Mahasiswa kerap bingung menentukan tugas mana yang harus diselesaikan lebih dulu ketika dihadapkan pada tenggat waktu bersamaan antara tugas mandiri, kuis, dan praktikum. Aplikasi to-do konvensional hanya mengurutkan secara alfabetis atau tanggal murni tanpa memahami bobot akademik dan kompleksitas tugas.
3. **Friksi Dinamika Kelompok (*The Free-Rider Dilemma*):** Kerja kelompok sering menjadi sumber stres akibat pembagian peran yang kabur, ketiadaan transparansi progres antar-anggota, dan komunikasi yang terfragmentasi di grup pesan instan.
4. **Toxic Gamification & Guilt-Inducing UX:** Sebagian besar aplikasi produktivitas komersial menggunakan alarm agresif dan badge merah yang mempermalukan pengguna ketika terlambat (*shame triggers*), alih-alih memberikan solusi pemulihan rencana yang realistis.
5. **Ketergantungan AI Paksa (*Forced AI Lock-in*):** Banyak aplikasi modern memaksa pengguna berlangganan layanan AI berbayar atau mewajibkan perangkat berspesifikasi tinggi hanya untuk mencatat to-do list harian.

---

## 2. Nilai Pokok Produk (Core Value Proposition)

**KerjaiAja** hadir sebagai platform produktivitas akademik hybrid yang dirancang oleh dan untuk pelajar dengan standar rekayasa modern yang inklusif, berdaulat, mandiri, dan berbiaya Rp0:

### A. Dual Experience Architecture
- **Mode A (Produktivitas Penuh Tanpa AI):**  
  Menjadi fondasi utama aplikasi. 100% fitur produktivitas berfungsi lengkap dan cepat tanpa membutuhkan model AI: manajemen tugas, mata kuliah, jadwal kuliah, kalender akademik, beban kerja, deterministik skor prioritas matematis (0–100), sinkronisasi cloud Supabase (jika login), dan kolaborasi tim. Ukuran download awal sangat ramping (~15–25 MB).
- **Mode B (Produktivitas dengan Asisten Belajar & Foto Soal On-Device Opsional):**  
  Bagi pengguna dengan perangkat yang memenuhi syarat hardware dan memilih untuk mengaktifkan AI, model Google Gemma (Gemma 3 1B IT / 270M) dijalankan langsung di perangkat via LiteRT-LM. Fitur Asisten Kerja berfokus pada tugas kontekstual (*Ask About My Work*, *Task Breakdown*, *Smart Planning*, *Contextual Recommendations*, *Work Review*) dan fitur akademik unggulan **Foto Soal & Pembahasan** (Jalur A: Google ML Kit OCR + Gemma Text Model + Validasi 7 Bagian).

### B. Empat Pilar Seimbang (The Balanced Quadriad)
1. *Personal Academic Productivity:* Pengelolaan tugas mandiri, mata kuliah/jadwal kuliah, pemecahan tugas mikro (*assignment breakdown*), pengatur waktu fokus, dan verifikasi berkas pengumpulan (*submission checklist*).
2. *Photo Question Solver & Learning Assistant:* Solusi pembahasan akademik terstruktur 7 bagian via foto dokumen tugas/soal untuk 12 rumpun mata pelajaran, didukung OCR on-device dan verifikasi jawaban deterministik.
3. *Intelligent & Explainable Prioritization:* Perhitungan skor prioritas tugas berbasis formula deterministik transparan (0–100) yang dapat diperkaya oleh penalaran kontekstual on-device Google Gemma, lengkap dengan penjelasan alasan (*why this task now*) dan kendali penuh pengguna (*user override*).
4. *Real-time Team Collaboration:* Ruang kerja kelompok akademik berbasis Supabase Realtime dan PostgreSQL RLS untuk pembagian tugas, pelacakan tanggung jawab anggota tim, dan sinkronisasi status pengerjaan secara instan.

### C. Filosofi Anti-Guilt UX
Sistem memperlakukan keterlambatan dan penundaan bukan sebagai kegagalan moral, melainkan sebagai deviasi jadwal yang wajar. Pengguna dibantu melalui opsi adaptif: *Reschedule with Assistant*, *Break into 15-Minute Micro Steps*, atau *Re-prioritize Queue*.

### D. Asisten Kerja Terarah & Bukan Chatbot Generik
AI dijalankan secara on-device di ponsel pengguna tanpa chatbot terbuka (*no generic chat*). AI hadir secara kontekstual hanya saat dibutuhkan sebagai mitra kerja untuk menstrukturkan pemikiran dan memecah tugas besar menjadi langkah realistis.

### E. Kedaulatan Privasi & Biaya Operasional Rp0
Data AI pengguna 100% diproses di perangkat lokal tanpa pernah dikirim ke server AI eksternal. Sinkronisasi cloud menggunakan Supabase Managed BaaS yang beroperasi penuh dalam batas aman Free Tier ($0).

---

## 3. Karakter & Prinsip Desain Produk

1. **Neo-Bauhaus: Form Follows Function:**  
   Tata letak geometris tegas, tipografi Inter berkontras tinggi, grid 12-kolom matematis, dan warna fungsional yang langsung mengomunikasikan prioritas serta status tanpa ornamen dekoratif yang sia-sia.
2. **Offline-First & Local-Write-First (Android Native):**  
   Aplikasi Android Native murni (Kotlin & Jetpack Compose) yang beroperasi secara offline menggunakan Room Database. Respon UI instan milidetik, sementara sinkronisasi cloud berjalan di background via WorkManager saat jaringan tersedia.
3. **Guest-First Onboarding:**  
   Pengguna dapat langsung menggunakan aplikasi tanpa membuat akun. Akun cloud Supabase bersifat opsional untuk sinkronisasi multi-perangkat dan kolaborasi kelompok, dengan migrasi data transparan.
4. **Satu Ekosistem Terpadu (Android Native + Web Admin CMS):**  
   Ekosistem KerjaiAja terdiri dari aplikasi Android untuk pelajar dan Web Admin CMS (Next.js 15) untuk pengelola sistem dalam mengatur silabus kurikulum dan menangani keluhan pengguna, didukung oleh satu basis data Supabase PostgreSQL bersama.

