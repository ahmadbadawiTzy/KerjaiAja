# Evaluasi Model On-Device: Google Gemma & Runtime LiteRT-LM — KerjaiAja
## Google Gemma 3 1B IT & Gemma 270M · LiteRT-LM Android SDK · Hardware Qualification

- **Tanggal Dokumen:** 2026-11-20
- **Status Evaluasi:** APPROVED BASELINE
- **Target Runtime:** Google AI Edge **LiteRT-LM** (Official Android / Kotlin API)
- **Lisensi Model:** Google Gemma Terms of Use (Open Weights)
- **Arsitektur:** Dual Experience (Mode A: Tanpa AI, Mode B: Asisten Kerja Opsional)

---

## 1. Tujuan & Metodologi Evaluasi

Evaluasi ini bertujuan memilih bobot model open-weight Google Gemma terbaik untuk dijalankan secara langsung di perangkat ponsel pengguna (*on-device AI*) tanpa server cloud:
1. **Intelligent Task Breakdown:** Memecah tugas besar menjadi 3–7 sub-tugas realistis.
2. **Contextual Task Prioritization:** Menganalisis urgensi kontekstual sebagai pelengkap Deterministic Engine.
3. **Deadline Risk Analysis:** Mengidentifikasi potensi keterlambatan berdasarkan beban dan riwayat tugas.
4. **Smart Planning:** Merekomendasikan alokasi sesi belajar harian.
5. **Contextual Blocker Detection:** Menemukan hambatan non-jadwal (ketergantungan mata kuliah/alat).

### Metodologi Pengujian
- **Dataset Evaluasi:** 50 skenario tugas akademik mahasiswa Indonesia (skripsi, praktikum informatika/teknik, tugas kelompok manajemen, esai sosiologi, dan ujian komprehensif).
- **Target Kuantisasi:** INT4 teroptimasi untuk LiteRT-LM.
- **Batasan Memori:** Maksimal konsumsi RAM runtime $\le 1.8\text{ GB}$ agar tidak memicu pemutusan paksa oleh Android Low Memory Killer (LMKD).

---

## 2. Matriks Komparatif Kandidat Model Gemma

| Parameter Evaluasi | **Gemma 3 1B IT**<br>*(Kandidat Utama)* | **Gemma 270M**<br>*(Kandidat Ultra-Ringan)* | **EmbeddingGemma 2**<br>*(Kandidat Semantic RAG)* |
| :--- | :---: | :---: | :---: |
| **Kategori Arsitektur** | Compact Instruction-Tuned Decoder | Nano Dense Transformer | Multimodal Embedding Vector Model |
| **Jumlah Parameter** | ~1.0 Miliar Parameter | ~270 Juta Parameter | ~300 Juta Parameter |
| **Ukuran Bobot Model (Disk)** | **~750 – 950 MB** (INT4) | **~250 – 350 MB** (INT4) | **~250 – 380 MB** (FP16/INT8) |
| **Konsumsi Puncak RAM** | **~850 MB – 1.15 GB** | **~350 – 500 MB** | **~450 – 600 MB** |
| **Dukungan Runtime Resmi** | LiteRT-LM (Android Native) | LiteRT-LM (Android Native) | LiteRT Embedding / MediaPipe |
| **Throughput CPU (tokens/s)** | 8 – 16 tokens/s *(GPU/NPU)* | 15 – 25 tokens/s *(CPU)* | N/A (Vector Generation) |
| **Cold Load Time (Disk ke RAM)**| < 2.0 detik | < 0.8 detik | < 0.6 detik |
| **Kualitas Bahasa Indonesia** | Sangat Baik (Penalaran Akademik) | Baik untuk instruksi pendek | Pemetaan Vektor Multibahasa |
| **JSON Schema Strictness** | $\ge 97\%$ | $\ge 88\%$ (Perlu Regex Sanitizer) | N/A |
| **Device Tier Target** | `RECOMMENDED` (RAM $\ge 6$ GB) | `LIMITED` (RAM 4 GB) | Evaluasi Pasca-MVP |

---

## 3. Evaluasi Mendalam Berdasarkan 5 Ranah Asisten Kerja

### 1. Kualitas Task Breakdown (Pemecahan Sub-Tugas)
- **Gemma 3 1B IT:** Unggul dalam memecah tugas akademik menjadi tahapan logis berurutan (misal: penentuan topik -> studi pustaka -> metodologi -> pengumpulan data -> pembahasan).
- **Gemma 270M:** Cukup baik untuk dekomposisi 3–4 langkah standar, namun membutuhkan prompt yang sangat ringkas dan template batasan ketat.

### 2. Ketepatan Analisis Prioritas (Priority Alignment)
- Kedua model tidak diizinkan menghitung angka prioritas (skor numerik 0–100 dihitung oleh Deterministic Priority Engine).
- **Gemma 3 1B IT** memberikan justifikasi kualitatif yang tajam mengapa suatu tugas harus didahulukan (misal: "Mata kuliah prasyarat skripsi berbobot 4 SKS").

### 3. Deadline Risk Detection (Deteksi Risiko Tenggat Waktu)
- **Gemma 3 1B IT** mampu mengaitkan selisih hari dengan estimasi beban jam kerja per sub-tugas secara konsisten.
- **Gemma 270M** cenderung hanya mengulang kembali tanggal tenggat waktu tanpa memberikan penilaian risiko beban kerja mendalam.

### 4. Kualitas Rekomendasi Aksi (Actionable Guidance)
- Rekomendasi dirancang dengan gaya komunikasi mahasiswa yang kompeten dan santai (*anti-guilt*).
- Menghindari kalimat yang memicu kecemasan (*shame triggers*).

### 5. Work Review (Pemeriksaan Kesiapan Pra-Pengumpulan)
- Memeriksa kesiapan checklist tugas sebelum pengumpulan.
- Format output divalidasi ke skema JSON secara ketat sebelum ditampilkan di modal preview.

---

## 4. Kebijakan Distribusi & Biaya Rp0

1. **Zero External AI API:** Dilarang menggunakan Gemini API, OpenAI API, Anthropic, atau Groq. Seluruh komputasi berjalan on-device.
2. **Dilarang Menyimpan di Supabase Storage:** Model weights wajib diunduh dari CDN resmi Google AI Edge / Hugging Face untuk melindungi kuota 1 GB Supabase Free Tier.
3. **Penyimpanan Sandbox Privat:** Berkas disimpan di `context.noBackupFilesDir/models/` dengan izin privat `0600` dan verifikasi SHA-256.
