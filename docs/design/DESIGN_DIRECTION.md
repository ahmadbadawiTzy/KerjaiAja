# Arah Desain Antarmuka (Design Direction) — KerjaiAja

Dokumen ini menjelaskan pedoman desain visual, sistem desain UI, dan prinsip pengalaman pengguna (*User Experience*) untuk **KerjaiAja**.

---

## 1. Filosofi & Karakter Desain

- **Karakter Produk:** **Profesional, Terfokus, dan Berwawasan Akademik.**
- **Anti-Gimmick & Anti-Slop:** Menghindari elemen gamifikasi kekanak-kanakan (karakter kartun berlebihan, efek suara berisik, atau lencana palsu) yang justru mengalihkan fokus belajar.
- **Tinggi Keterbacaan (*High Legibility*):** Tata letak dioptimalkan untuk membaca teks instruksi tugas panjang, tenggat waktu, dan checklist terstruktur dengan cepat.
- **Ergonomi Satu Tangan (*One-Hand Usability*):** Komponen interaksi utama (tombol tambah tugas, filter, navigasi) ditempatkan di zona jangkauan jempol bawah layar (*bottom-weighted navigation*).

---

## 2. Sistem Desain: Google Material 3 (Material You)

Aplikasi dibangun penuh di atas spesifikasi **Material Design 3 (M3)**:

### 2.1 Palet Warna & Warna Dinamis (Dynamic Color)
- Mendukung fitur **Dynamic Theming** (Material You) pada Android 12+ (API 31+), menyelaraskan palet warna aplikasi dengan wallpaper pengguna.
- Untuk Android 10–11 (API 29–30), aplikasi menggunakan palet warna standar berbasis nada biru-keabuan (*academic slate blue*) yang tenang dan berwibawa:
  - **Primary:** Deep Slate Blue (`#1E3A8A` / `#93C5FD` di Dark Mode)
  - **Secondary:** Muted Teal (`#0D9488` / `#5EEAD4` di Dark Mode)
  - **Surface & Background:** Bersih dengan elevasi tonal halus tanpa bayangan tebal.
  - **Error & Urgency:** Coral Red terkalibrasi (`#DC2626` / `#F87171`) khusus untuk penanda deadline mendesak.

### 2.2 Tipografi
- Menggunakan font sistem Android modern yang bersih dan proporsional (**Roboto / Google Sans**).
- Skala tipe M3 yang disiplin:
  - `Headline Small/Medium`: Judul layar dan nama mata kuliah.
  - `Title Medium`: Judul tugas dan kartu utama.
  - `Body Medium/Large`: Teks instruksi tugas dan materi catatan.
  - `Label Large/Small`: Tag mata kuliah, indikator status, dan badge deadline.

### 2.3 Dark Mode
- Dukungan penuh untuk mode gelap (*Dark Theme*) dengan warna latar abu-abu gelap pekat (bukan hitam absolut OLED menyilaukan) untuk kenyamanan belajar di malam hari tanpa kelelahan mata (*eye strain*).

---

## 3. Komponen Utama & Interaksi Mikro

1. **Task Card yang Terstruktur:**  
   Kartu tugas menampilkan informasi penting secara ringkas: kode mata kuliah, judul tugas, tenggat waktu (dengan warna adaptif sesuai urgensi), dan bilah progres sub-tugas.
2. **Breakdown Stepper Interaktif:**  
   Visualisasi berurutan untuk sub-langkah tugas, dengan *checkbox* animasi halus yang memberikan kepuasan psikologis saat menyelesaikan langkah kerja.
3. **Floating Bottom Navigation & Action Bar:**  
   Navigasi bawah yang ramping untuk perpindahan cepat antara Tugas, Mata Kuliah, Jadwal Belajar, dan Asisten AI.
4. **Aksesibilitas (Accessibility Standards):**
   - Target sentuh minimal: **48 x 48 dp** untuk semua tombol dan ikon interaktif.
   - Rasio kontras teks minimal **4.5:1** terhadap latar belakang sesuai pedoman WCAG AA.
