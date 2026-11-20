# Strategi Evaluasi & Benchmark Khusus AI On-Device (On-Device AI Evaluation Strategy) — KerjaiAja
## Google Gemma 3 1B IT & Gemma 270M · LiteRT-LM Android SDK · Hardware Qualification

Dokumen ini mendefinisikan kerangka kerja evaluasi teknis, batas Service Level Agreement (SLA), serta spesifikasi **KerjaiAja On-Device Academic Benchmark Suite** untuk menguji dan membandingkan performa model **Google Gemma (Gemma 3 1B IT dan Gemma 270M)** menggunakan runtime **Google AI Edge LiteRT-LM** di Android Native pasca-rebaseline arsitektur hybrid.

---

## 1. Kondisi & Standar Pengujian On-Device

1. **Dataset Pengujian Identik:** Seluruh kandidat dievaluasi menggunakan *KerjaiAja Indonesian Academic Benchmark Set* (50 skenario tugas akademik mahasiswa: tugas praktikum coding, makalah riset, kuis, proyek kelompok, dan persiapan ujian).
2. **Lingkungan Hardware Mobile:** Model diuji pada perangkat Android fisik dengan spesifikasi berjenjang:
   - *Tier LIMITED:* RAM fisik 4 GB, Chipset entry/mid-range (Snapdragon 4/6 series), CPU 2-4 thread.
   - *Tier RECOMMENDED:* RAM fisik $\ge 6\text{ GB}$, Chipset Snapdragon 7/8 series atau Google Tensor (akselerasi GPU/NPU).
3. **Runtime & Parameter:**
   - Mesin inferensi: **LiteRT-LM** (Google AI Edge Android SDK).
   - Context window: 1024 – 2048 tokens.
   - Kuantisasi bobot: INT4 quantized.
   - Temperature: `0.2` (untuk output terstruktur deterministik).
   - Max generation tokens: `350` tokens.

---

## 2. Matriks Pengukuran 11 Dimensi Benchmark

```text
┌────────────────────────────────────────────────────────────────────────┐
│               KERJAIAJA ON-DEVICE 11-DIMENSION BENCHMARK               │
├──────────────────────────────────┬─────────────────────────────────────┤
│ 1. Kualitas Task Breakdown       │ 7. Total Inference Latency          │
│ 2. Spearman Correlation Prioritas│ 8. Peak RAM Consumption (LMKD Risk) │
│ 3. Urgency Classification F1     │ 9. Battery Impact per Query         │
│ 4. Hidden Barrier Detection      │ 10. Model Cold Load Time            │
│ 5. Bahasa Indonesia & Nuansa     │ 11. Device Tiering Compatibility    │
│ 6. JSON Strict Validity Rate     │                                     │
└──────────────────────────────────┴─────────────────────────────────────┘
```

### 2.1 Target Kelulusan Produksi vs Profil Model

| Dimensi Pengujian | Target Produksi Minimum | **Gemma 3 1B IT** *(Kandidat Utama)* | **Gemma 270M** *(Kandidat Ultra-Ringan)* | Catatan Pengujian |
| :--- | :---: | :---: | :---: | :--- |
| **1. Kualitas Task Breakdown** | **$\ge 85.0\%$** | **Est: $88 - 92\%$** | Est: $78 - 82\%$ | Logika tahapan sub-tugas berurutan. |
| **2. Spearman Correlation ($\rho$)** | **$\ge 0.85$** | **Est: $0.86 - 0.89$** | Est: $0.80 - 0.83$ | Diuji terhadap prioritas pakar akademik. |
| **3. Urgency Classification F1** | **$\ge 0.88$** | **Est: $0.90 - 0.93$** | Est: $0.82 - 0.85$ | Klasifikasi 4 kuadran urgensi. |
| **4. Hidden Barrier Detection** | **$\ge 80.0\%$** | **Est: $84 - 88\%$** | Est: $70 - 75\%$ | Deteksi kebutuhan lab/survei lapangan. |
| **5. Bahasa Indonesia & Nuansa** | **$\ge 85.0\%$** | **Est: $89 - 93\%$** | Est: $80 - 84\%$ | Istilah SKS, praktikum, bahasa kampus. |
| **6. JSON Strict Validity Rate** | **$\ge 98.0\%$** | **Est: $\ge 98.0\%$** | Est: $88.0 - 92.0\%$ | Parsing data class `kotlinx.serialization`. |
| **7. Total Latency (300 tokens)** | $\le 3.5\text{ detik}$ | **$\sim 2.0 - 3.5\text{ detik}$** | **$\sim 1.5 - 2.5\text{ detik}$** | Diuji pada akselerasi CPU/GPU mobile. |
| **8. Peak RAM Consumption** | $\le 1.85\text{ GB}$ | **$\sim 850 - 1.150\text{ MB}$** | **$\sim 350 - 500\text{ MB}$** | Batas toleransi aman Android LMKD. |
| **9. Battery Impact per Query** | $\le 0.08\%$ | **$\sim 0.03 - 0.05\%$** | **$\sim 0.01 - 0.02\%$** | Transient run selama 2-4 detik. |
| **10. Model Cold Load Time** | $\le 2.5\text{ detik}$ | **$\sim 1.4 - 2.0\text{ detik}$** | **$\sim 0.6 - 0.9\text{ detik}$** | Memory-mapped I/O dari penyimpanan disk. |
| **11. Device Compatibility** | Mid & High Tiers | **`RECOMMENDED` (RAM $\ge 6$ GB)** | **`LIMITED` (RAM 4 GB)** | RAM $< 4$ GB dialihkan ke Mode A (Tanpa AI). |

---

## 3. Protokol Validasi Fisik & Aturan Gerbang Keputusan

1. **Pengujian Bertingkat pada Smartphone Nyata:**  
   Benchmark wajib dijalankan pada minimal 2 perangkat fisik:
   - 1 unit smartphone kelas menengah (misal: Snapdragon 695 / 778G, RAM 6 GB).
   - 1 unit smartphone kelas atas (misal: Snapdragon 8 Gen 2 / Tensor G3, RAM 8–12 GB).
2. **Kriteria Kelulusan Mutlak (Zero Crash / Zero LMK):**  
   Jika model memicu crash OOM atau dihentikan paksa oleh sistem operasi Android dalam pengujian berulang (10x run beruntun), model tersebut diblokir untuk kelas perangkat tersebut oleh `DeviceCapabilityManager`.
3. **Kesiapan Fallback Deterministik:**  
   Jika validitas JSON berada di bawah 98%, sistem otomatis beralih ke template pemecahan tugas manual agar pengguna tidak pernah mendapati tampilan error.
