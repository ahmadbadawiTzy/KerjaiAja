# Spesifikasi Fitur Foto Soal & Pembahasan (Photo Question Solver Specification) — KerjaiAja
## Pembahasan Akademik Edukatif · On-Device Processing · 12 Rumpun Mata Pelajaran

Dokumen ini mendefinisikan spesifikasi teknis lengkap untuk fitur **Foto Soal & Pembahasan (Photo Question Solver)** pada platform KerjaiAja.

---

## 1. Tujuan & Filosofi Desain

Fitur Foto Soal dirancang sebagai **pendamping belajar edukatif (*learning companion*)**, bukan mesin contekan instan:
- **Proses Berpikir yang Jelas:** Membimbing siswa memahami struktur soal, mengidentifikasi data penting, menentukan rumus/konsep yang relevan, dan menuntun langkah demi langkah pengerjaan hingga jawaban akhir.
- **100% On-Device:** Seluruh proses citra, OCR, inferensi AI, dan validasi jawaban dieksekusi secara lokal di ponsel pengguna tanpa internet dan tanpa server eksternal.
- **Human-in-the-Loop:** Menampilkan hasil ekstraksi teks OCR untuk diverifikasi dan dikoreksi pengguna sebelum model AI menghasilkan pembahasan.

---

## 2. Cakupan 12 Rumpun Materi Akademik

Setiap mata pelajaran diproses dengan karakteristik prompt dan validasi yang disesuaikan:

| Rumpun Pelajaran | Karakteristik Pembahasan Khusus | Format Wajib dalam Langkah |
| :--- | :--- | :--- |
| **1. Matematika** | Ekstraksi simbol/notasi, substitusi variabel, verifikasi konsistensi | Rumus, substitusi angka, hitungan bertahap, alternatif cek |
| **2. Fisika** | Identifikasi besaran fisis, konversi satuan SI, diagram gaya | Data diketahui, konversi satuan, rumus fisika, cek kewajaran |
| **3. Kimia** | Stoikiometri, persamaan reaksi setara, nomor atom/massa | Persamaan reaksi, mol/konsentrasi, kalkulasi, satuan |
| **4. Biologi** | Konsep taksonomi, mekanisme fisiologis, interaksi ekosistem | Terminologi ilmiah, hubungan sebab-akibat, poin penjelasan |
| **5. Bahasa Indonesia** | Analisis ide pokok, koherensi paragraf, kaidah tata bahasa EYD | Kutipan bukti paragraf, kaidah bahasa, simpulan |
| **6. Bahasa Inggris** | Reading comprehension, grammar rule, tense alignment | Context clues, grammar justification, target vocabulary |
| **7. Sejarah** | Kronologi peristiwa, latar belakang kausal, signifikansi sejarah | Rentang waktu/tokoh, faktor penyebab, dampak sejarah |
| **8. Geografi** | Fenomena geosfer, prinsip geografi, dinamika lingkungan | Pendekatan geografi, faktor spasial, penjelasan fenomena |
| **9. Ekonomi** | Konsep pasar, elastisitas, akuntansi, penalaran kebijakan | Formula ekonomi, tabel kalkulasi (jika ada), analisis logis |
| **10. Sosiologi** | Gejala sosial, interaksi masyarakat, teori struktural | Teori sosiologis relevan, analisis fakta sosial |
| **11. PPKn** | Prinsip konstitusi, hierarki norma hukum, hak & kewajiban | Rujukan pasal/norma, penegakan etika, argumen hukum |
| **12. Akademik Lainnya** | Penyesuaian fleksibel berbasis teks instruksi soal | Penjelasan terstruktur umum, konsep kunci, solusi |

---

## 3. Format Skema Output Wajib (7 Bagian)

Model AI on-device diwajibkan mengeluarkan JSON terstruktur yang diparsing ke data class Kotlin `StructuredSolution`:

```json
{
  "subject": "MATHEMATICS",
  "question_identification": "Soal mencari himpunan penyelesaian dari Sistem Persamaan Linear Dua Variabel (SPLDV) dengan metode eliminasi/substitusi.",
  "given_information": [
    "Persamaan (1): 2x + 3y = 12",
    "Persamaan (2): 4x - y = 10"
  ],
  "asked_objective": "Nilai variabel x dan y serta nilai dari x + 2y",
  "concept_and_formulas": [
    "Metode Eliminasi: menyamakan koefisien salah satu variabel",
    "Metode Substitusi: memasukkan nilai variabel yang ditemukan ke persamaan lain"
  ],
  "step_by_step_solution": [
    {
      "step_number": 1,
      "title": "Eliminasi variabel y",
      "explanation": "Kalikan persamaan (2) dengan 3 agar koefisien y berlawanan tanda dengan persamaan (1): 3 * (4x - y = 10) menjadi 12x - 3y = 30.",
      "math_expression": "12x - 3y = 30"
    },
    {
      "step_number": 2,
      "title": "Jumlahkan kedua persamaan",
      "explanation": "(2x + 3y) + (12x - 3y) = 12 + 30 -> 14x = 42 -> x = 3.",
      "math_expression": "x = 42 / 14 = 3"
    },
    {
      "step_number": 3,
      "title": "Substitusi nilai x = 3 ke persamaan (2)",
      "explanation": "4(3) - y = 10 -> 12 - y = 10 -> y = 2.",
      "math_expression": "y = 12 - 10 = 2"
    },
    {
      "step_number": 4,
      "title": "Hitung nilai x + 2y",
      "explanation": "Substitusikan x = 3 dan y = 2 ke dalam ekspresi yang ditanyakan: 3 + 2(2) = 3 + 4 = 7.",
      "math_expression": "3 + 2(2) = 7"
    }
  ],
  "final_answer": "Nilai x = 3, y = 2, dan hasil dari x + 2y = 7.",
  "verification_check": "Uji pada persamaan (1): 2(3) + 3(2) = 6 + 6 = 12 (Terbukti benar).",
  "confidence_score": 0.98,
  "uncertainty_note": null
}
```

---

## 4. State Machine Antarmuka Pengguna (UI States)

Alur interaksi pada Jetpack Compose diatur melalui state machine eksplisit:
1. `IDLE`: Menunggu input (tombol Buka Kamera / Pilih Galeri).
2. `IMAGE_SELECTED`: Gambar terpilih, layar menampilkan antarmuka Crop, Rotate, dan Retake.
3. `PREPROCESSING`: Pengecekan kualitas gambar lokal (deteksi blur, rotasi otomatis).
4. `OCR_PROCESSING`: Google ML Kit mengekstraksi teks soal di perangkat (<300 ms).
5. `OCR_REVIEW`: Teks hasil ekstraksi ditampilkan dalam text editor agar pengguna dapat memverifikasi atau mengoreksi angka/simbol rumus.
6. `AI_PROCESSING`: Inferensi model Gemma on-device memproses prompt terstruktur.
7. `VALIDATING`: `AnswerValidator` memvalidasi aritmetika deterministik dan format skema.
8. `SUCCESS`: Pembahasan 7 bagian ditampilkan rapi dengan kartu interaktif.
9. `PARTIAL_RESULT`: Hasil ditampilkan dengan badge peringatan ambiguitas jika OCR memiliki keyakinan sedang.
10. `ERROR`: Penanganan kesalahan transparan (misal: gambar tidak memuat teks atau model timeout).
11. `UNSUPPORTED_INPUT`: Gambar diagram visual murni di perangkat yang belum mendukung model vision.
