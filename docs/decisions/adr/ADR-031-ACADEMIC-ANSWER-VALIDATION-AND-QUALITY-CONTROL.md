# ADR-031: Lapisan Validasi Jawaban Akademik & Kontrol Kualitas (Academic Answer Validation and Quality Control)

## Status
Accepted

## Konteks
Model bahasa besar (LLM) secara inheren bersifat probabilistik dan rentan terhadap halusinasi faktual, kesalahan komputasi aritmetika sederhana, sitasi fiktif, atau penjelasan yang terdengar meyakinkan namun salah secara substansi (*hallucination / false confidence*). Dalam aplikasi edukasi, menyajikan jawaban yang salah kepada siswa dapat berakibat fatal bagi pemahaman konsep dan nilai akademik mereka. Oleh karena itu, sistem tidak boleh menyerahkan hasil mentah LLM langsung ke pengguna tanpa melalui lapisan validasi independen.

## Keputusan
Kami menetapkan **Lapisan Validasi Jawaban Akademik Independen (Academic Answer Validation Layer)** yang memvalidasi hasil inferensi model sebelum ditampilkan kepada pengguna:

### 1. Validasi Deterministik Aritmetika & Rumus (Math & Science Validator)
- Untuk mata pelajaran kuantitatif (Matematika, Fisika, Kimia):
  - Sistem mengekstraksi ekspresi aritmetika pada langkah pengerjaan dan memvalidasi kebenaran kalkulasi menggunakan engine evaluasi deterministik Kotlin murni.
  - Pengecekan konsistensi dimensi satuan (misal: kecepatan harus $\text{m/s}$, energi dalam $\text{Joule}$, massa dalam $\text{kg}$).
  - Jika kalkulasi model salah, sistem mengoreksi angka akhir secara deterministik pada langkah pengerjaan atau memberi catatan koreksi.

### 2. Validasi Struktur Skema JSON (Strict JSON Validation & Bounded Auto-Repair)
- Respons model diwajibkan mematuhi skema JSON 7 bagian (`StructuredSolution`).
- Menggunakan `kotlinx.serialization` dengan batas 1x *bounded regex auto-repair* untuk membersihkan karakter non-JSON di awal/akhir respons.
- Jika JSON tetap gagal diparsing: sistem **TIDAK MENAMPILKAN DATA RUSAK**, melainkan menampilkan pesan kegagalan secara transparan dengan opsi *Coba Ulang* (*Retry*).

### 3. Validasi Konsistensi Logika & Fakta (Fact & Uncertainty Scoring)
- Untuk soal objektif (Pilihan Ganda): Memastikan jawaban akhir memilih opsi yang tersedia (A, B, C, D, atau E) dengan alasan yang koheren.
- Menilai tingkat kepastian (*confidence indicator*): Jika teks OCR memiliki keraguan atau soal ambigu, sistem secara eksplisit menampilkan teks catatan: *"Perhatian: Teks soal memiliki ambiguitas pada bagian X. Pembahasan ini mengasumsikan Y."*
- **Larangan Sitasi Fiktif:** Model dilarang mengarang nama buku paket, pasal undang-undang palsu, atau nama jurnal fiktif.

### 4. Benchmark Terpisah per Mata Pelajaran (Multi-Subject Benchmark)
- Kualitas model dievaluasi secara independen pada 12 mata pelajaran menggunakan dataset akademik Indonesia. Dilarang mengklaim seluruh mata pelajaran berhasil hanya berdasarkan pengujian pada soal matematika.

## Konsekuensi
### Positif
- Integritas edukatif terjamin: siswa mendapatkan penjelasan yang teruji dan valid.
- Menghilangkan "keyakinan palsu" (*false certainty*) pada soal yang tidak jelas.
- Mendeteksi dan memperbaiki kesalahan hitung sepele model secara deterministik.

### Negatif / Mitigasi
- Latensi pemrosesan bertambah ~50–150 ms untuk validasi lokal (masih jauh di bawah batas toleransi pengguna).
