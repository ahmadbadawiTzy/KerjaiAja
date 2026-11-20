# Strategi On-Device Optical Character Recognition (OCR)
## KerjaiAja — Human-in-the-Loop OCR Architecture

Dokumen ini menjelaskan strategi teknis pemilihan, evaluasi, integrasi, dan mitigasi keterbatasan Optical Character Recognition (OCR) *on-device* untuk fitur **Foto Soal & Pembahasan** pada KerjaiAja.

---

## 1. Rasional Pemilihan OCR-First (Jalur A)

Dalam merancang fitur Foto Soal & Pembahasan untuk ekosistem perangkat pelajar Indonesia (yang didominasi smartphone kelas menengah ke bawah dengan RAM 3 GB – 6 GB), KerjaiAja menetapkan pendekatan **OCR-First** sebagai jalur pemrosesan utama (*Jalur A*).

### Perbandingan Karakteristik: Jalur A (OCR-First) vs Jalur B (End-to-End Multimodal Vision)

| Parameter Teknis | Jalur A: OCR-First (ML Kit + Gemma Text) | Jalur B: Vision Multimodal (Gemma 3n E2B) |
| :--- | :--- | :--- |
| **Footprint Memori (RAM)** | $\mathbf{< 35\text{ MB}}$ untuk OCR engine | $\mathbf{> 2.2\text{ GB}}$ untuk Vision Transformer |
| **Latensi Ekstraksi** | $\mathbf{150 - 350\text{ ms}}$ di CPU Android | $\mathbf{4.000 - 12.000\text{ ms}}$ di GPU/NPU |
| **Ukuran Download Awal** | $\mathbf{\sim 7\text{ MB}}$ (Model ML Kit terkompresi) | $\mathbf{\sim 1.5 - 2.0\text{ GB}}$ bobot model |
| **Kompatibilitas Perangkat**| Universal (Android 8.0+, RAM $\ge 3\text{ GB}$) | Terbatas (Flagship/Upper Mid-Range, RAM $\ge 8\text{ GB}$) |
| **Verifiabilitas Teks** | **Sangat Tinggi** (Siswa dapat membaca & mengedit teks) | **Rendah** (Kotak hitam/black-box inferensi) |
| **Risiko Halusinasi** | Rendah (Konteks teks terverifikasi sebelum masuk LLM) | Tinggi jika gambar buram/glare |

---

## 2. Engine OCR Terpilih: Google ML Kit Text Recognition v2

KerjaiAja mengadopsi library resmi:
```toml
# gradle/libs.versions.toml
mlkit-text-recognition = { module = "com.google.mlkit:text-recognition", version = "16.0.1" }
```

### Keunggulan Google ML Kit Text Recognition v2:
1. **100% On-Device & Offline**: Ekstraksi teks berjalan lokal tanpa transmisi data ke internet.
2. **Didukung Google Play Services**: Binary dan model bahasa Latin didistribusikan melalui Play Services, menjaga ukuran dasar APK tetap ramping.
3. **Akurasi Teks Cetak Bahasa Indonesia & Inggris**: Sangat presisi membaca buku teks pelajaran, lembar kerja siswa (LKS), modul perkuliahan, dan soal ujian cetak.
4. **Struktur Hierarkis Blok**: Menyediakan hirarki `Text -> TextBlock -> Line -> Element/Word` yang memudahkan rekonstruksi paragraf, daftar pilihan ganda (A, B, C, D, E), dan nomor soal.

---

## 3. Keterbatasan OCR & Strategi Mitigasi

Meskipun Google ML Kit sangat andal membaca teks latin standar, terdapat beberapa keterbatasan inheren pada pembacaan materi akademik eksakta:

### Keterbatasan 1: Simbol & Rumus Matematika Bersusun
- **Karakteristik Masalah**: Pecahan bertingkat ($\frac{a}{b}$), akar kuadrat bertingkat ($\sqrt[3]{x}$), integral ($\int_a^b$), dan matriks sering kali terbaca sebagai pecahan baris yang terpisah-pisah atau simbol acak.
- **Solusi Mitigasi (Human-in-the-Loop Review Screen)**:
  - Sebelum dikirim ke model AI, hasil ekstraksi OCR wajib disajikan dalam layar peninjauan (*Review Screen*).
  - Disediakan **Academic Quick-Toolbar** yang berisi tombol cepat simbol akademik:
    - Simbol Aritmetika & Aljabar: `²`, `³`, `√`, `±`, `÷`, `×`, `≠`, `≤`, `≥`, `∞`, `π`
    - Notasi Fisika/Kimia: `Δ`, `α`, `β`, `θ`, `λ`, `Ω`, `→`, `⇌`
    - Template Notasi: Pecahan `[a/b]`, Pangkat `x^[y]`, Indeks `x_[n]`
  - Siswa dapat mengoreksi angka atau variabel yang salah dibaca dalam hitungan detik.

### Keterbatasan 2: Tulisan Tangan Siswa (Handwriting)
- **Karakteristik Masalah**: Variasi ketebalan pena, kemiringan tulisan tangan, dan coretan koreksi menghasilkan akurasi OCR yang lebih rendah dibandingkan teks cetak.
- **Solusi Mitigasi**:
  - Deteksi probabilitas keyakinan (*confidence score*).
  - Tampilkan pesan edukatif: *"Tulisan tangan terdeteksi. Silakan periksa kembali kata-kata yang kami tandai dengan warna kuning agar AI dapat memberikan jawaban akurat."*

### Keterbatasan 3: Soal Bergambar & Grafik Sains
- **Karakteristik Masalah**: OCR teks tidak dapat membaca diagram sirkuit listrik, kurva trigonometri, bangun ruang 3D, atau bagan reaksi kimia.
- **Solusi Mitigasi**:
  - Jika pengguna menandai soal memiliki diagram yang tidak terbaca teks:
    - Opsi A (Perangkat Standar): Pengguna dipandu mendeskripsikan informasi diagram secara teks di kolom deskripsi tambahan (misal: *"Sebuah segitiga siku-siku ABC di B dengan panjang AB = 3 cm dan BC = 4 cm"*).
    - Opsi B (Perangkat Kelas Tinggi / Vision Qualified): Tawarkan Jalur B (Inference Multimodal Vision Lab) jika pengguna telah mengaktifkan model multimodal opsional.

---

## 4. Confidence Scoring & Threshold Policy

ML Kit menghasilkan nilai `confidence` pada tingkat kata dan baris. KerjaiAja menerapkan aturan penandaan berikut:

```kotlin
data class OcrEvaluationResult(
    val extractedText: String,
    val averageConfidence: Float,
    val lowConfidenceWords: List<String>,
    val needsManualReview: Boolean
)

fun evaluateOcrConfidence(lines: List<Text.Line>): OcrEvaluationResult {
    var totalConfidence = 0f
    var elementCount = 0
    val lowConfidenceWords = mutableListOf<String>()

    for (line in lines) {
        for (element in line.elements) {
            val conf = element.confidence ?: 1.0f
            totalConfidence += conf
            elementCount++
            if (conf < 0.65f) {
                lowConfidenceWords.add(element.text)
            }
        }
    }

    val avgConfidence = if (elementCount > 0) totalConfidence / elementCount else 0f
    val needsReview = avgConfidence < 0.75f || lowConfidenceWords.isNotEmpty()

    return OcrEvaluationResult(
        extractedText = lines.joinToString("\n") { it.text },
        averageConfidence = avgConfidence,
        lowConfidenceWords = lowConfidenceWords,
        needsManualReview = needsReview
    )
}
```

- **Confidence $\ge 0.85$**: Status teks *Sangat Jelas* (tombol "Lanjutkan ke Pembahasan" langsung aktif dengan badge hijau).
- **Confidence $0.65 - 0.84$**: Status teks *Cukup Jelas* (kata dengan confidence rendah ditandai garis bawah kuning).
- **Confidence $< 0.65$**: Status teks *Kurang Jelas* (muncul banner dialog peringatan: *"Foto kurang fokus atau tulisan terpotong. Disarankan mengambil ulang foto atau melengkapi teks secara manual."*).

---

## 5. Penolakan OCR Berbasis Cloud (Zero Cloud OCR)

KerjaiAja secara tegas **menolak** penggunaan Cloud OCR API eksternal (seperti Google Cloud Vision API, Azure Computer Vision, atau AWS Textract) berdasarkan empat pilar kebijakan:
1. **Zero External Paid API**: Mencegah tagihan berulang (*pay-per-request*) yang membebani siswa atau pengembang.
2. **Kedaulatan Privasi Pelajar**: Lembar tugas siswa dan materi ujian internal sekolah tidak boleh ditransmisikan ke server analitik pihak ketiga.
3. **Kemampuan Offline Penuh**: Siswa di wilayah minim sinyal atau yang mematikan data seluler tetap dapat menggunakan fitur OCR 100% normal.
4. **Keamanan Tanpa Token**: Tidak ada risiko kebocoran API Key di dalam binary APK Android yang bisa direverse-engineer pihak tidak bertanggung jawab.
