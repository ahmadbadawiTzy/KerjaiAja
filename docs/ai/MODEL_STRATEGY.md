# Strategi Pemilihan Model AI On-Device (On-Device Model Strategy) — KerjaiAja
## Google Gemma 3 1B IT & Gemma 270M · Hardware Tiering · Zero External AI API

Dokumen ini mendefinisikan strategi pemilihan model on-device (**On-Device AI Model Strategy**), kebutuhan sumber daya perangkat bergerak, dan tata kelola distribusi model pada platform **KerjaiAja** pasca-rebaseline arsitektur hybrid.

---

## 1. Filosofi & Batasan AI On-Device

1. **Dual Experience:** Mode A (Produktivitas Penuh Tanpa AI) adalah fondasi utama; Mode B (Asisten Kerja On-Device Gemma) adalah kapabilitas tambahan opsional bagi perangkat yang memenuhi syarat.
2. **Target Biaya Rp0 & Privasi Penuh:** Tidak ada server inferensi cloud, tidak ada panggilan ke Gemini API, OpenAI API, Anthropic, atau Groq. Nol byte data tugas yang dikirim ke cloud untuk AI.
3. **Bukan Chatbot Generik:** AI difokuskan secara kontekstual pada 5 use case produktivitas (*Ask About My Work*, *Task Breakdown*, *Smart Planning*, *Contextual Recommendations*, *Work Review*).
4. **Pemisahan Bobot Model & Zero Supabase Storage:** Model bobot **DILARANG** disimpan di Supabase Storage untuk menjaga kuota Free Tier ($0). Bobot model diunduh langsung dari CDN resmi Google AI Edge / Hugging Face.

---

## 2. Matriks Kandidat Model Google Gemma

```text
┌────────────────────────────────────────────────────────────────────────┐
│                      KERJAIAJA ON-DEVICE MODEL MATRIX                  │
├──────────────────────────┬───────────┬──────────────┬──────────────────┤
│ Model                    │ Parameter │ Format       │ Peran Sistem     │
├──────────────────────────┼───────────┼──────────────┼──────────────────┤
│ 1. Gemma 3 1B IT         │ ~1.0 B    │ .litertlm    │ Kandidat Utama   │
│ 2. Gemma 270M            │ ~270 M    │ .litertlm    │ Kandidat Ringan  │
│ 3. EmbeddingGemma 2      │ ~300 M    │ .tflite      │ Evaluasi RAG V2  │
└──────────────────────────┴───────────┴──────────────┴──────────────────┘
```

### 2.1 Google Gemma 3 1B IT — Kandidat Utama (Primary Target Candidate)
- **Karakteristik:** Compact Instruction-Tuned Transformer dari Google DeepMind.
- **Format:** LiteRT-LM (`.litertlm` quantized INT4).
- **Rasional Pemilihan:**
  - Kemampuan penalaran instruksi akademik sangat baik dalam bahasa Indonesia.
  - Kepatuhan skema JSON terstruktur $\ge 97\%$.
  - Ukuran berkas terkuantisasi INT4 berkisar **~750 – 950 MB**, dengan konsumsi RAM runtime **~850 MB – 1.15 GB**.
  - Ditargetkan untuk perangkat kategori **`RECOMMENDED`** (RAM fisik $\ge 6\text{ GB}$, RAM bebas $\ge 2.5\text{ GB}$).

### 2.2 Google Gemma 270M — Kandidat Ultra-Ringan (Ultra-Lightweight Candidate)
- **Karakteristik:** Dense Nano Transformer 270 Juta Parameter.
- **Rasional Pemilihan:**
  - Ditujukan untuk perangkat kategori **`LIMITED`** (RAM fisik 4 GB, RAM bebas $1.2 - 2.5\text{ GB}$).
  - Ukuran biner sangat kecil (**~250 – 350 MB**) dengan konsumsi RAM runtime **~350 – 500 MB**.
  - Mengizinkan mahasiswa dengan ponsel entry-level untuk menjalankan dekomposisi tugas dasar tanpa risiko OOM crash.

### 2.3 EmbeddingGemma 2 — Kandidat Semantic Retrieval (Evaluasi Tahap 2)
- **Karakteristik:** Dense embedding model ~300M parameter untuk vector search.
- **Rasional Pemilihan:**
  - Dievaluasi untuk pencarian semantik tingkat lanjut pasca-MVP v1.0.
  - Pada MVP v1.0, sistem mengandalkan **SQLite FTS5 BM25** yang efisien, instan (< 25 ms), dan tidak menambah beban RAM.

---

## 3. Matriks Kebutuhan Sumber Daya Perangkat (Hardware Footprint)

| Parameter Perangkat | Mode A (Tanpa AI) | Mode B: Gemma 270M (`LIMITED`) | Mode B: Gemma 3 1B (`RECOMMENDED`) |
| :--- | :---: | :---: | :---: |
| **RAM Fisik Minimum** | **2 GB RAM** | **4 GB RAM** | **6 GB RAM** |
| **RAM Bebas Minimum** | $\ge 200\text{ MB}$ | $\ge 1.2\text{ GB}$ | $\ge 2.5\text{ GB}$ |
| **Ruang Disk Bebas** | $\ge 100\text{ MB}$ | $\ge 2.0\text{ GB}$ | $\ge 3.5\text{ GB}$ |
| **Akselerator CPU/GPU** | CPU murni | CPU (2–4 threads) | GPU (OpenCL/Vulkan) / NPU |
| **Konsumsi Puncak RAM** | ~70–90 MB | ~450 MB | ~1.150 MB |
| **Durasi Inferensi** | < 1 ms (Instant) | ~1.5 – 3.0 detik | ~2.0 – 4.0 detik |

---

## 4. Kebijakan Distribusi & Tata Kelola Model

1. **Zero External AI API:** Dilarang menggunakan Gemini API, OpenAI API, Anthropic API, Groq, atau server AI eksternal apa pun.
2. **Penyimpanan Sandbox Terisolasi:** Berkas model disimpan di direktori privat aplikasi `context.noBackupFilesDir/models/` dengan validasi checksum SHA-256.
3. **Hak Kendali Pengguna:** Pengguna dapat mengunduh atau menghapus model sewaktu-waktu di menu Pengaturan tanpa mengganggu data tugas pribadi.

---

## 5. Integrasi Pipeline Progresif & Tata Kelola Memori

Sesuai dengan **ADR-033** dan **ADR-034**:

1. **Pemanggilan Model Hanyalah Tahap Akhir (Stage 5):**
   Model AI tidak pernah dipanggil untuk sorting, validasi form, atau dekomposisi berbasis aturan deterministik. Lebih dari 90% interaksi diselesaikan di Stage 0–2.
2. **Single Active Session Mutex:**
   Model LiteRT-LM diisolasi di balik `Mutex`. Hanya satu sesi inferensi yang dapat aktif pada satu waktu. Jika ada permintaan kedua saat inferensi berlangsung, permintaan tersebut ditolak dengan status sibuk atau diantrekan secara elegan.
3. **Lazy Loading & Auto-Unload:**
   Model tidak pernah dimuat di memori saat boot aplikasi. Instance model dilepaskan secara otomatis jika tidak ada aktivitas selama 5 menit (`idleTimeoutMillis = 300_000L`).
4. **Respon Cepat LMK:**
   Model segera melepaskan bobot saat menerima sinyal `onTrimMemory(TRIM_MEMORY_RUNNING_CRITICAL)` untuk mencegah terminasi paksa OS.

---

## 6. Referensi Putusan Arsitektur Terkait

- [ADR-014: On-Device AI via LiteRT-LM & Google Gemma](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-014-ON-DEVICE-AI-LITERTLM-GEMMA.md)
- [ADR-022: Photo Question Solver Architecture with Dual Pipeline](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-022-PHOTO-QUESTION-SOLVER-ARCHITECTURE.md)
- [ADR-033: Progressive AI Processing Pipeline](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-033-PROGRESSIVE-AI-PROCESSING-PIPELINE.md)
- [ADR-034: On-Device Model Lifecycle & Memory Management](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-034-ON-DEVICE-MODEL-LIFECYCLE-AND-MEMORY-MANAGEMENT.md)
- [ADR-035: Per-Model Device Capability Evaluation](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-035-PER-MODEL-DEVICE-CAPABILITY-EVALUATION.md)

