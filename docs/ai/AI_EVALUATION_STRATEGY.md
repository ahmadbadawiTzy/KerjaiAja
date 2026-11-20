# Strategi Evaluasi AI (AI Evaluation Strategy) — KerjaiAja

Dokumen ini mendefinisikan kerangka kerja evaluasi teknis dan kualitas output untuk **Adaptive Local AI Engine** pada **KerjaiAja**.

---

## 1. Metrik Kinerja Inferensi On-Device (Hardware & Runtime Metrics)

Evaluasi performa lokal wajib diukur secara empiris pada perangkat fisik nyata (*real device testing*), bukan semata-mata pada emulator Android Studio:

| Metrik | Definisi & Pengukuran | Target Kinerja Minimum (SLA) |
| :--- | :--- | :--- |
| **Time to First Token (TTFT)** | Waktu dari tombol breakdown ditekan hingga token respons pertama keluar. | <= 2.5 detik pada Snapdragon 7-series / Dimensity 7000 |
| **Generation Throughput (TPS)** | Kecepatan keluaran token rata-rata (*Tokens Per Second*). | >= 8 – 12 tokens/detik (nyaman dibaca streaming) |
| **Peak Memory Footprint (RAM)** | Lonjakan alokasi memori runtime saat inferensi berlangsung. | <= 1.2 GB tambahan di atas base RAM aplikasi |
| **Thermal & Battery Impact** | Kenaikan suhu baterai dan konsumsi daya selama 5 sesi inferensi beruntun. | Kenaikan suhu <= 2.5°C; Tanpa *thermal throttling* ekstrim |
| **Cold Model Load Time** | Waktu yang dibutuhkan untuk memuat bobot dari storage ke RAM. | <= 3.0 detik |

---

## 2. Metrik Kualitas Output (Output Quality & Schema Metrics)

Karena keluaran AI digunakan untuk mengisi sub-tugas dan checklist secara otomatis di aplikasi, kualitas parsing dan kepatuhan skema (*schema adherence*) menjadi sangat kritikal:

### 2.1 Kepatuhan Skema JSON (JSON Validity Rate)
- **Target:** >= 98% keberhasilan parsing JSON valid.
- Respons model harus langsung dapat diparsing oleh serializer Kotlin (`kotlinx.serialization`) tanpa ada karakter rusak atau markdown syntax yang tidak teratur.

### 2.2 Relevansi Dekonstruksi Tugas (Actionability Score)
- Sub-tugas yang dihasilkan harus konkret, realistis, dan berurutan secara logis (misal: "1. Tentukan topik riset", "2. Cari 3 jurnal referensi", bukan "1. Kerjakan semua tugas").
- Pengujian dilakukan menggunakan dataset evaluasi yang mencakup 50 ragam tugas akademik umum di Indonesia (makalah, laporan praktikum, presentasi kelompok, resume buku).

### 2.3 Pencegahan Halusinasi & Batasan Peran (Guardrails Adherence)
- Model tidak boleh memberikan jawaban esai utuh yang melanggar integritas akademik (mencegah *plagiarism enabler*).
- Model wajib menolak instruksi di luar lingkup penataan tugas akademik.

---

## 3. Protokol Benchmarking Perangkat Nyata (Real-Device Benchmark Protocol)

Benchmarking dilakukan pada tiga tingkatan perangkat target:
1. **Tier Rendah (Low-End Baseline):** 4 GB RAM, chipset entry-level (misal: Helio G85 / Snapdragon 680). Fokus: Uji kestabilan memori dan ketahanan LMK (*Low Memory Killer*).
2. **Tier Menengah (Mid-Range Sweet Spot):** 6–8 GB RAM, chipset mid-range (misal: Dimensity 7050 / Snapdragon 7s Gen 2). Fokus: Evaluasi TTFT dan TPS optimal.
3. **Tier Atas (High-End Reference):** 8–12 GB RAM, flagship chipset (Snapdragon 8 Gen 2/3). Fokus: Komparasi performa LiteRT-LM dengan ML Kit Gemini Nano.
