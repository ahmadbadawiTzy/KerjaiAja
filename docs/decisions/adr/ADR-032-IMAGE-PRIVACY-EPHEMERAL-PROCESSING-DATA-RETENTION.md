# ADR-032: Privasi Citra Soal, Pemrosesan Lokal Efemeral & Retensi Data (Image Privacy, Ephemeral Processing & Data Retention)

## Status
Accepted

## Konteks
Fitur Foto Soal & Pembahasan melibatkan pengambilan foto dokumen akademik, buku, lembar catatan pribadi, atau tugas sekolah yang berpotensi memuat informasi sensitif (nama siswa, NIM/NIS, nama guru/dosen, foto teman sekelas, atau lokasi sekolah). Pengunggahan foto secara diam-diam ke cloud atau penyimpanan jangka panjang yang tidak terkontrol melanggar prinsip *Privacy-by-Design*, bertentangan dengan Google Play Store Data Safety, serta menghabiskan ruang penyimpanan lokal dan kuota cloud pengguna.

## Keputusan
Kami menetapkan kebijakan **Privasi Citra Soal, Pemrosesan Lokal Efemeral, dan Retensi Data**:

### 1. Default Behavior: Pemrosesan Efemeral Lokal 100%
- Foto soal diproses secara lokal di memori sandbox privat perangkat.
- **DILARANG MENGUNGGAH FOTO SOAL KE CLOUD SECARA OTOMATIS:** Foto tidak pernah dikirim ke Supabase Storage, database cloud, atau layanan analitik eksternal mana pun saat proses pemecahan soal berlangsung.
- File citra sementara (*temporary cache files*) yang dibuat saat kamera mengambil gambar atau pemotongan (*crop*) **wajib dibersihkan seketika** dari cache disk setelah inferensi selesai atau dibatalkan oleh pengguna.

### 2. Hak Simpan & Pemisahan Hasil Pembahasan
- Pengguna diberikan kendali penuh melalui tindakan eksplisit:
  1. *Hanya Salin Teks Jawaban:* Mengambil teks pembahasan tanpa menyimpan foto.
  2. *Simpan Pembahasan ke Catatan Belajar Lokal:* Menyimpan teks pembahasan terstruktur ke database Room lokal.
  3. *Simpan Foto Soal (Opsional):* Foto soal hanya disimpan secara lokal jika pengguna secara sadar mencentang opsi *"Simpan Gambar Soal Bersama Pembahasan"*.
  4. *Hapus Instan:* Tombol hapus menghapus berkas gambar dan entitas pembahasan seketika tanpa residu (*zero retention residual*).

### 3. Larangan Penggunaan Data untuk AI Training
- Gambar soal, teks hasil OCR, dan hasil pembahasan pengguna **DILARANG KERAS DIGUNAKAN UNTUK TRAINING MODEL AI**, perbaikan dataset publik, atau monetisasi data pihak ketiga.

### 4. Sanitasi Log & Logging Hygiene
- Isi teks soal dan path gambar dilarang dicetak ke dalam Android Logcat pada build release.
- Jika terjadi error pada pipeline OCR atau AI, log hanya mencatat kode error dan dimensi citra (panjang x lebar) tanpa isi konten teks soal.

## Konsekuensi
### Positif
- Kepatuhan mutlak terhadap Google Play Store Data Safety Policy dan regulasi perlindungan data pribadi (GDPR & UU PDP Indonesia).
- Ruang penyimpanan ponsel pengguna tidak membengkak oleh tumpukan file cache foto sementara.
- Kepercayaan penuh dari pelajar, orang tua, dan institusi pendidikan.

### Negatif / Mitigasi
- Jika pengguna tidak sengaja menutup aplikasi sebelum menekan simpan, foto sementara terhapus (dimitigasi dengan konfirmasi modal saat pengguna menekan tombol kembali).
