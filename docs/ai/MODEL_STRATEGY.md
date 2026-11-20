# Strategi Model AI (Model Strategy) — KerjaiAja

Dokumen ini menjelaskan strategi pemilihan model bahasa alami, runtime eksekusi, serta metodologi adaptasi kecerdasan buatan on-device untuk **KerjaiAja**.

---

## 1. Pemilihan Model & Runtime

### 1.1 Kandidat Model Utama: Gemma 3 1B IT
- **Karakter:** Model berukuran 1 miliar parameter (*1 Billion parameter instruction-tuned*) dari Google DeepMind.
- **Justifikasi Pemilihan:**
  - Menawarkan rasio penalaran terhadap konsumsi memori yang sangat kompetitif untuk tugas-tugas penstrukturan, peringkasan, dan pembuatan daftar (*task decomposition & outlining*).
  - Bobot terkuantisasi (4-bit / INT4) memiliki ukuran berkas sekitar 600–900 MB, memungkinkan pengunduhan on-demand yang realistis pada perangkat seluler.
  - Memiliki pemahaman instruksi bahasa Indonesia dan Inggris yang memadai untuk domain akademik.

### 1.2 Kandidat Runtime Inferensi: LiteRT-LM
- **Platform:** Google AI Edge LiteRT (sebelumnya TensorFlow Lite) khusus Large Language Models.
- **Justifikasi Pemilihan:**
  - Integrasi native dengan ekosistem Android Native (C++/Java/Kotlin bindings).
  - Dukungan akselerasi perangkat keras via GPU (OpenCL/Vulkan) dan CPU multithreading modern.
  - Alokasi memori yang terprediksi dan efisien untuk perangkat *memory-constrained*.

### 1.3 Alternatif Pendukung: Gemini Nano via ML Kit GenAI
- Pada perangkat flagship/terbaru yang memiliki dukungan native Android AICore (misal: Pixel, Galaxy kelas atas), aplikasi dapat secara opsional memanfaatkan API **Gemini Nano** via ML Kit GenAI, menghilangkan kebutuhan untuk mengunduh bobot model mandiri.

---

## 2. Matriks Keputusan: Terkonfirmasi vs. Asumsi Riset

Penting untuk memisahkan keputusan arsitektural yang telah terkonfirmasi dari asumsi yang masih membutuhkan pengujian perangkat fisik (*device benchmarking*):

| Aspek / Parameter | Status Keputusan | Detail & Catatan Validasi |
| :--- | :---: | :--- |
| **Arsitektur On-Device** | ✅ **Terkonfirmasi** | Inferensi wajib berjalan lokal di perangkat pengguna. |
| **Tanpa Kewajiban Cloud AI API** | ✅ **Terkonfirmasi** | Tidak ada API OpenAI/Claude berbayar yang wajib bagi pengguna. |
| **Tanpa Server VPS Inferensi** | ✅ **Terkonfirmasi** | Tidak menyewa atau mewajibkan server backend untuk pemrosesan LLM. |
| **Aplikasi Tetap Fungsional Tanpa AI** | ✅ **Terkonfirmasi** | Fitur inti non-AI (Task, Subject, Checklist) 100% mandiri. |
| **Min Android API 29 (Android 10)** | ✅ **Terkonfirmasi** | Batas minimum SDK untuk kompatibilitas runtime modern. |
| **Prioritas Prompt Eng. & RAG** | ✅ **Terkonfirmasi** | Menolak fine-tuning di awal; memaksimalkan SQLite FTS + Context. |
| **Kelayakan Kuantisasi Gemma 3 1B IT** | 🔬 *Asumsi / Perlu Riset* | Perlu diuji: apakah kuantisasi 4-bit mempertahankan akurasi format JSON sub-tugas. |
| **Stabilitas RAM 4–6 GB** | 🔬 *Asumsi / Perlu Riset* | Perlu diuji di perangkat fisik 4GB RAM: apakah background process OS memicu LMK (Low Memory Killer). |
| **Kecepatan Inferensi (Tokens/sec)** | 🔬 *Asumsi / Perlu Riset* | Target >= 8-12 tokens/detik pada chipset mid-range (misal: Snapdragon 6/7 series atau Dimensity 700/8000). |
| **Dukungan ML Kit GenAI Gemini Nano** | 🔬 *Asumsi / Perlu Riset* | Aksesibilitas API Gemini Nano pada perangkat non-Pixel masih terbatas dan bervariasi per vendor. |

---

## 3. Metodologi Adaptasi: Prompt Engineering & Local RAG

Pendekatan adaptasi kecerdasan buatan dilakukan melalui **rekayasa prompt terstruktur** dan **Retrieval-Augmented Generation (RAG) lokal**, bukan fine-tuning bobot model:

1. **Rekayasa Prompt Terstruktur:**
   - Menyediakan template prompt sistem (*system prompt*) dengan peran yang sangat spesifik: *"Asisten pemecah tugas akademik"*.
   - Mewajibkan format keluaran dalam skema JSON yang kaku (*constrained decoding* atau *JSON schema enforcement*), memudahkan parsing ke objek Kotlin.
2. **Local RAG via SQLite FTS:**
   - Mengambil intisari catatan materi yang tersimpan di Room database lokal.
   - Menggabungkan materi tersebut sebagai blok konteks `[KONTEKS MATERI]` di dalam prompt, sehingga rekomendasi sub-tugas relevan dengan silabus mata kuliah tanpa memperbesar ukuran model.
3. **Penolakan Fine-Tuning Dini:**
   - Fine-tuning membutuhkan dataset khusus berskala besar, biaya komputasi tinggi, dan menyulitkan distribusi pembaruan model. Strategi ini ditolak hingga efektivitas RAG lokal terbukti tidak mencukupi.
