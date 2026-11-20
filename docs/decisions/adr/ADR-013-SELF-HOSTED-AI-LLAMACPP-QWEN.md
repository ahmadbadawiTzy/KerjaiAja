# ADR-013: Self-Hosted AI Architecture with llama.cpp & Resource-Optimized Model Evaluation

## Status
ACCEPTED (2026-11-20, Diperbarui untuk Evaluasi Optimalisasi Resource & Biaya Server)

## Konteks
Pada perancangan awal arsitektur backend inferensi mandiri, Qwen3-4B ditetapkan sebagai kandidat model primer. Namun, dalam rangka menekan kebutuhan sumber daya perangkat keras server (*hardware footprint*) dan meminimalkan biaya operasional bulanan (*operational expenditure*) agar aplikasi dapat di-hosting pada VPS berspesifikasi ekonomis (misal: VPS 4 GB RAM dengan biaya ~$4–$8/bulan), diperlukan evaluasi ulang strategi model secara empiris.

Tantangan utama yang dihadapi:
1. **Trade-off Kebutuhan RAM vs Penalaran:** Model 4B membutuhkan ~3.2 GB RAM (model + KV cache), yang menyulitkan penerapan pada VPS 4 GB RAM bersamaan dengan Go server dan PostgreSQL.
2. **Kualitas Bahasa Indonesia & Struktur JSON:** Model yang terlalu kecil (< 1B) sering kali mengalami degradasi penalaran konteks tugas kompleks dan kegagalan kepatuhan skema JSON.
3. **Kedaulatan & Kerahasiaan Data Tetap Mutlak:** Dilarang menggunakan API pihak ketiga (Gemini, OpenAI, Anthropic, Groq, dll.). Seluruh inferensi harus berjalan pada model yang di-host sendiri via `llama.cpp` atau runtime yang terverifikasi.

## Keputusan
KerjaiAja menetapkan strategi **Multi-Candidate Resource-Optimized Evaluation** dengan tetap mempertahankan **Qwen3-4B GGUF Q4_K_M sebagai baseline kualitas**:

### 1. Hierarki Kandidat Model
1. **`Qwen3-1.7B GGUF Q4_K_M` — Kandidat Utama Baru (New Primary Candidate):**
   - Ditargetkan sebagai kandidat utama baru untuk rilis produksi MVP berbiaya hemat.
   - Ukuran model di disk ~1.10 GB; alokasi RAM + KV cache (4K context) ~1.6 GB.
   - Dapat beroperasi sangat stabil pada server VPS 4 GB RAM dengan kecepatan generasi token tinggi (35–50 tokens/s pada CPU).
2. **`Qwen3-0.6B GGUF Q4_K_M` — Kandidat Ultra-Ringan (Ultra-Lightweight Candidate):**
   - Ukuran model di disk ~420 MB; alokasi RAM + KV cache ~700 MB.
   - Ditujukan untuk pengujian batas bawah (*lower-bound exploration*) pada server mikro 1–2 GB RAM.
3. **`LFM2.5-1.2B GGUF Q4_K_M` — Kandidat Pembanding (Comparative Candidate):**
   - Model Liquid Foundation / arsitektur hybrid kompak ~1.2B parameter.
   - Diuji sebagai pembanding arsitektural non-dense standar terhadap famili Qwen untuk efisiensi context memory.
4. **`Qwen3-4B GGUF Q4_K_M` — Baseline Kualitas (Quality Baseline):**
   - **Dipertahankan sebagai baseline acuan kualitas tertinggi.**
   - Model ini menjadi tolok ukur batas kelulusan bagi seluruh model alternatif yang lebih kecil.

### 2. Standar Runtime Inferensi: llama.cpp
- Seluruh kandidat wajib dijalankan menggunakan runtime internal **`llama.cpp`** (C++20 binary) atau runtime kompatibel GGUF yang tervalidasi.
- Berjalan sebagai layanan privat internal di localhost (`127.0.0.1:8081`) dan **TIDAK PERNAH** diekspos ke public internet.

### 3. Ketentuan Mutlak & Larangan AI Eksternal
- **TIDAK MENGGUNAKAN** Gemini API, OpenAI API, Anthropic API, Groq API, Hugging Face Inference API, atau layanan AI eksternal berbayar/gratis lainnya.
- **TIDAK MENGUBAH** Go + Gin sebagai backend utama.
- **TIDAK MEMINDAHKAN** inferensi ke perangkat Android (tetap di-host di server).
- **TIDAK MENGHAPUS** *Deterministic Prioritization Engine* (tetap berdiri independen sebagai komponen terpisah).
- **TIDAK MENGUNDUH** model besar sebelum persetujuan eksplisit.
- **TIDAK MELAKUKAN** deployment atau fine-tuning prematur.

### 4. Ambang Batas Kelulusan Produksi (Production Acceptance Thresholds)
Model yang lebih kecil (Qwen3-1.7B, Qwen3-0.6B, LFM2.5-1.2B) **HANYA BOLEH MENGGANTIKAN QWEN3-4B SECARA RESMI DI PRODUKSI** apabila berhasil lulus seluruh kriteria gerbang kualitas berikut pada *KerjaiAja Benchmark Suite*:
- **Korelasi Prioritas Spearman ($\rho$):** $\ge 0.85$ terhadap ground truth penilaian akademik.
- **F1-Score Urgency Classification:** $\ge 0.88$ pada 4 kategori (*Critical, High, Medium, Low*).
- **Kepatuhan Skema JSON:** $\ge 98\%$ tanpa error parsing pada percobaan pertama.
- **Actionability & Explainability:** Penjelasan logis transparan dan sub-tugas konkret tanpa halusinasi.

## Konsekuensi
### Positif
- Potensi penurunan kebutuhan RAM server hingga 50–70% (dari 8 GB ke 4 GB RAM), memangkas biaya sewa VPS bulanan hingga separuhnya (~$4–$10/bulan).
- Peningkatan throughput komputasi token per detik (TPS) hingga 2x lebih cepat pada CPU server hemat.
- Keputusan pemilihan model didasarkan pada data empiris nyata, bukan sekadar asumsi ukuran parameter.

### Negatif / Mitigasi
- Memerlukan eksekusi benchmark komparatif yang cermat terhadap 4 kandidat sebelum penetapan rilis final.
  - *Mitigasi:* Menggunakan dataset evaluasi dan kondisi pengujian yang seragam pada `docs/ai/AI_EVALUATION_STRATEGY.md`.
