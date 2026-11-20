# ADR-028: Arsitektur Pemrosesan Soal OCR-First (OCR-First Question Processing Architecture)

## Status
Accepted

## Konteks
Fitur Foto Soal & Pembahasan memerlukan konversi visual teks soal fisik (buku paket, lembar ujian, kertas coretan, atau tangkapan layar) menjadi data teks yang dapat dipahami oleh model AI. Model multimodal vision membutuhkan sumber daya memori yang sangat besar (~1.5–2.5 GB RAM) dan komputasi berat, yang sering kali tidak dapat dijalankan pada smartphone kelas menengah atau low-end. Sebagian besar soal akademik (khususnya soal teks naratif, esai, pilihan ganda, dan sebagian besar soal sains berformat teks) dapat diekstraksi menggunakan Optical Character Recognition (OCR) berbobot ringan.

## Keputusan
Kami mengadopsi arsitektur **Jalur A (OCR-First Architecture)** sebagai alur utama default pada MVP:

### 1. Komponen Pipeline OCR-First
1. **Input Citra:** Kamera langsung atau Galeri perangkat pengguna.
2. **Pra-Pemrosesan Citra Lokal (On-Device Image Preprocessing):**
   - Crop, rotate, auto-contrast, dan binarization grayscale ringan.
   - Pengecekan kualitas citra (*Image Quality Check*): deteksi blur ekstrem atau pencahayaan terlalu gelap.
3. **On-Device OCR Engine (Google ML Kit Text Recognition v2):**
   - Pustaka: `com.google.mlkit:text-recognition:16.0.1` (Android on-device).
   - Ekstraksi teks latin dan angka berkecepatan tinggi (< 300 ms) langsung di perangkat tanpa koneksi internet.
4. **Segmentasi & Deteksi Multiple Questions:** Memisahkan nomor soal (misal: "1.", "2.a", "No. 3") jika gambar memuat lebih dari satu soal.
5. **Review & Koreksi Manual Pengguna (Human-in-the-Loop OCR Review):**
   - UI menampilkan teks hasil ekstraksi sebelum inferensi AI dijalankan.
   - Pengguna dapat menyunting karakter, angka, atau simbol rumus yang mungkin salah terbaca oleh OCR.
6. **Grounding & Inferensi Gemma Text Model:** Teks yang telah terverifikasi diteruskan ke model bahasa Google Gemma (Gemma 3 1B / Gemma 270M) bersama Local RAG referensi akademik pengguna.

### 2. Batasan Terbuka & Transparansi OCR
- OCR umum memiliki keterbatasan dalam mengenali formula matematika bersusun kompleks (akar pangkat rumit, integral lipat, matriks multi-kolom) serta simbol non-standar.
- Jika tingkat keyakinan (*confidence score*) OCR di bawah ambang batas aman (< 70%), sistem **DILARANG MENGARANG JAWABAN DENGAN KEYAKINAN PALSU**. Sistem wajib meminta pengguna untuk mengambil foto ulang atau memperbaiki teks secara manual di layar editor OCR.

## Konsekuensi
### Positif
- Sangat ringan di memori RAM (< 80 MB saat OCR berjalan).
- Kompatibel dengan model teks yang sudah ada (Gemma 3 1B IT dan Gemma 270M).
- Tingkat akurasi tinggi pada soal berbasis teks, esai, dan pilihan ganda.
- Koreksi manual oleh pengguna mencegah halusinasi fatal pada langkah pengerjaan awal.

### Negatif / Mitigasi
- Kurang optimal untuk soal yang jawabannya sangat bergantung pada interpretasi visual geometri, grafik kurva, diagram biologis, atau rangkaian listrik (dimitigasi dengan Jalur B Multimodal Vision Model sebagai ekstensi terpisah pada perangkat yang kompatibel).
