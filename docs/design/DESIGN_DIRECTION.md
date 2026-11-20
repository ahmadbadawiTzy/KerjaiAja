# Arah Desain Antarmuka (Design Direction) — KerjaiAja

Dokumen ini menetapkan arah visi visual, prinsip estetika, dan pendekatan pengalaman pengguna (*User Experience*) untuk **KerjaiAja** pasca-rebaseline arsitektur penuh (Full Architecture Rebaseline 2026-11-20), mengadopsi bahasa desain **Neo-Bauhaus × Contemporary Gen Z × Professional Academic Productivity**.

---

## 1. Filosofi & Identitas Visual

KerjaiAja memadukan ketegasan fungsional dari gerakan Bauhaus abad ke-20 dengan kepekaan estetika kontemporer generasi Z yang dinamis:

1. **Form Follows Function (Bentuk Mengikuti Fungsi):**  
   Setiap elemen visual dirancang untuk tujuan tertentu: mengomunikasikan batas waktu tugas, menjelaskan alasan prioritas, dan mempermudah kolaborasi kelompok. Desain menolak keras ornamen dangkal atau animasi lambat yang menghambat produktivitas.
2. **Anti-Gimmick & Anti-Guilt:**  
   Menolak sistem gamifikasi manipulatif (seperti maskot yang sedih ketika pengguna tidak membuka aplikasi) dan alarm histeris yang memicu rasa bersalah (*academic anxiety*). KerjaiAja adalah alat bantu kerja yang menghargai kesehatan mental mahasiswa.
3. **Keterbacaan Ekstrem (Extreme Legibility):**  
   Mengutamakan tipografi Inter dengan kontras tinggi, hierarki ukuran tegas, dan angka tabular untuk timer dan batas waktu, menjamin kenyamanan navigasi cepat di bawah terik matahari kampus maupun di ruang perpustakaan temaram.
4. **Sentuhan Kontemporer (Neo-Bauhaus Energy):**  
   Memadukan kanvas warm off-white yang nyaman dengan aksen warna primer berani (*Electric Cobalt Blue*, *Bauhaus Amber*, dan *Crimson Urgency*) serta kontur border geometris yang tegas.

---

## 2. Pilar Pengalaman Pengguna (UX Pillars)

### 2.1 Ergonomi Satu Tangan (One-Hand Bottom Ergonomics)
Seluruh tombol aksi utama (tombol tambah tugas, filter urgensi, navigasi antar pilar) diposisikan di zona jangkauan jempol bawah layar (*bottom-weighted interaction*), memudahkan pengoperasian satu tangan saat mahasiswa sedang berjalan di lorong kampus.

### 2.2 Umpan Balik Taktil yang Memuaskan (Satisfying Micro-Interactions)
- Saat sub-tugas ditandai selesai, sistem memberikan respons haptik mikro (*haptic click*) dan animasi garis coret (*strikethrough*) geometris instan.
- Bilah progres tugas diperbarui secara halus dengan transisi kurva interpolasi Material 3 FastOutSlowIn.

### 2.3 Transparansi Penalaran (Explainable & Contextual)
Antarmuka tidak pernah menampilkan angka prioritas misterius tanpa keterangan. Mahasiswa dapat menyentuh badge urgensi untuk melihat kartu penjelasan yang menguraikan bobot waktu, bobot nilai, dan catatan AI dalam 3 baris poin yang mudah dicerna.

---

## 3. Sistem Desain Rinci & Integrasi Skill

Spesifikasi detail mengenai token warna, skala tipografi Inter, ukuran grid, bentuk sudut, serta implementasi komponen Jetpack Compose diatur secara komprehensif dalam:
- [`docs/design/DESIGN_SYSTEM.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/design/DESIGN_SYSTEM.md)
- Skill lokal: `.agents/skills/bauhaus-design/`
