# ADR-027: Spesifikasi Fitur Foto Soal & Pembahasan Akademik (Photo Question Solver)

## Status
Accepted

## Konteks
Sebagai platform produktivitas dan pendamping belajar (*academic productivity and learning companion*), pelajar dan mahasiswa tidak hanya menghadapi tantangan administratif manajemen tugas, melainkan juga kesulitan substantif saat memahami soal dan materi pelajaran (PR, latihan ujian, tugas praktikum). Menyediakan fitur pemecahan soal berbasis foto (*photo question solver*) akan meningkatkan nilai guna KerjaiAja secara signifikan. Namun, fitur ini tidak boleh sekadar menjadi "mesin contekan jawaban instan", melainkan alat bantu belajar yang menyajikan proses pemikiran dan pembahasan langkah demi langkah secara mendalam, transparan, dan edukatif.

## Keputusan
Kami menetapkan fitur baru **Foto Soal & Pembahasan (Photo Question Solver)** dengan ketentuan arsitektural sebagai berikut:

### 1. Ketersediaan Bersyarat (Conditional Feature Gate)
- Fitur ini **HANYA AKTIF** pada **Mode B (Productivity with On-Device AI)** ketika pengguna memilih untuk memasang model AI yang kompatibel pada perangkat yang memenuhi syarat hardware.
- Pengguna pada Mode A (Tanpa AI) tetap menikmati 100% fitur produktivitas personal, deterministik prioritas, dan sinkronisasi tanpa terganggu atau diblokir.

### 2. Cakupan 12 Materi Pelajaran Akademik
Fitur dirancang untuk mendukung 12 rumpun mata pelajaran dengan karakteristik pembahasan spesifik:
1. Matematika (simbol, rumus, substitusi, kalkulasi bertahap, pemeriksaan konsistensi).
2. Fisika (data, satuan, konversi, prinsip dasar, kalkulasi, validasi kewajaran).
3. Kimia (stoikiometri, reaksi kimia, konsentrasi, konsep molekuler).
4. Biologi (konsep biologis, klasifikasi, mekanisme sebab-akibat sistem hayati).
5. Bahasa Indonesia (analisis paragraf, kaidah EYD, konteks bacaan, penjelasan logis).
6. Bahasa Inggris (grammar, reading comprehension, vocabulary, context analysis).
7. Sejarah (kronologi, konteks historis, faktor kausalitas, fakta terverifikasi).
8. Geografi (analisis spasial, fenomena geosfer, korelasi iklim/lingkungan).
9. Ekonomi (konsep elastisitas, perhitungan akuntansi/pasar, penalaran ekonomi).
10. Sosiologi (struktur sosial, interaksi masyarakat, teori sosiologis).
11. Pendidikan Pancasila & Kewarganegaraan (prinsip konstitusi, norma hukum, etika).
12. Materi Akademik Lainnya.

### 3. Struktur Wajib Output Pembahasan Edukatif
Hasil pembahasan harus memenuhi 7 bagian terstruktur:
1. **Identifikasi Soal:** Pemahaman sistem terhadap tipe dan inti pertanyaan.
2. **Informasi yang Diketahui:** Ekstraksi fakta, angka, variabel, atau data dari teks soal.
3. **Yang Ditanyakan:** Penegasan target capaian soal.
4. **Konsep / Rumus yang Digunakan:** Teori atau formula relevan.
5. **Langkah Pengerjaan:** Urutan pengerjaan logis bertahap.
6. **Jawaban Akhir:** Hasil final yang jelas dan presisi.
7. **Pemeriksaan & Verifikasi:** Pengecekan ulang kalkulasi atau konsistensi jawaban.

### 4. Zero Cloud Inference & Zero Automatic Cloud Upload
- Gambar diproses **100% lokal** di perangkat Android pengguna.
- Foto dan hasil pembahasan **DILARANG** diunggah otomatis ke Supabase Storage atau cloud server. Foto hanya disimpan ke memori lokal jika pengguna secara sadar menekan tombol simpan.

## Konsekuensi
### Positif
- KerjaiAja berevolusi menjadi asisten akademik komprehensif (produktivitas + pemahaman materi).
- Pendekatan langkah demi langkah mendidik mahasiswa alih-alih memicu ketergantungan instan.
- Privasi terlindungi mutlak dan biaya server tetap Rp 0.

### Negatif / Mitigasi
- Membutuhkan penanganan pipeline citra dan OCR yang teliti (dimitigasi dengan Jalur A OCR-first dan UI koreksi teks manual).
- Keragaman format soal membutuhkan validator output yang adaptif (dimitigasi dengan `AnswerValidator` modular).
