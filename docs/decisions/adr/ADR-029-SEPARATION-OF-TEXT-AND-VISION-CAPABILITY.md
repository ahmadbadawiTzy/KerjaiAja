# ADR-029: Pemisahan Evaluasi Kapabilitas Model Text vs Model Vision (Separation of Text and Vision Model Capabilities)

## Status
Accepted

## Konteks
Kebutuhan komputasi model bahasa murni (Text-Only LLM seperti Gemma 3 270M atau Gemma 3 1B) sangat berbeda secara fundamental dibandingkan model multimodal visual (Vision LLM seperti Gemma 3n E2B atau model visual transformer lainnya). Model multimodal membutuhkan pemrosesan token gambar (*image patch embeddings*), konteks window memori yang jauh lebih besar, dan alokasi RAM yang mencapai 1.8–2.5 GB. Menyamaratakan status kualifikasi perangkat untuk seluruh model AI adalah kesalahan rekayasa yang dapat memicu crash Out-of-Memory (OOM) atau melarang perangkat mid-range menjalankan fitur teks yang sebenarnya mampu mereka jalankan.

## Keputusan
Kami menetapkan **Pemisahan Tegas Evaluasi Kapabilitas Model Text vs Model Vision** di seluruh arsitektur sistem dan modul [`DeviceCapabilityManager`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/app/src/main/java/com/kerjaiaja/app/core/device/DeviceCapabilityManager.kt):

### 1. Evaluasi Kapabilitas Terpisah & Independen
Satu perangkat fisik dapat memiliki status kelayakan yang berbeda untuk setiap kategori model:
- **`TextCapability`:**
  - Evaluasi kelayakan untuk menjalankan `Gemma 3 270M` (RAM bebas $\ge 1.2\text{ GB}$) atau `Gemma 3 1B IT` (RAM bebas $\ge 2.5\text{ GB}$).
  - Membuka akses untuk Asisten Kerja dan Foto Soal berbasis OCR-First (Jalur A).
- **`VisionCapability`:**
  - Evaluasi kelayakan untuk menjalankan model multimodal `Gemma 3n E2B` (RAM fisik $\ge 8\text{ GB}$, RAM bebas $\ge 3.5\text{ GB}$, storage bebas $\ge 6.0\text{ GB}$, akselerasi GPU/NPU aktif via OpenCL/Vulkan/NNAPI).
  - Membuka akses untuk Jalur B (penalaran langsung pada diagram, grafik, geometri visual).

### 2. Matriks Kompatibilitas Fitur vs Model

| Kapabilitas Fitur | Text Model (Gemma 3 1B / 270M) | Vision Model (Gemma 3n E2B) |
| :--- | :---: | :---: |
| **Task Breakdown & Asisten Kerja** | Ya (Wajib) | Tidak Diperlukan (Overkill) |
| **Foto Soal Teks via OCR** | Ya (Jalur Utama) | Bisa Langsung |
| **Interpretasi Diagram Geometri** | Terbatas (Hanya Teks Deskriptif) | Ya (Jika Lolos Uji Lab) |
| **Analisis Grafik & Kurva Sains** | Terbatas | Ya (Jika Lolos Uji Lab) |
| **Penyelesaian Rumus Bertingkat** | Ya (Setelah Koreksi Teks OCR) | Ya |

### 3. Kebijakan Unduhan Model Terpisah (Independent Downloads)
- Pengguna yang perangkatnya mendukung kedua kategori model diberikan kebebasan untuk:
  1. Hanya mengunduh Text Model (menghemat storage disk ponsel).
  2. Mengunduh Text Model + Vision Model secara terpisah.
- Tidak ada bundling paksa berkas model biner.

## Konsekuensi
### Positif
- Menghindari diskriminasi perangkat: perangkat dengan RAM 4–6 GB tetap dapat menikmati Asisten Kerja dan Foto Soal teks via OCR.
- Mengeliminasi risiko crash OOM akibat memuat model multimodal di perangkat yang tidak memadai.
- Memberikan transparansi mutlak kepada pengguna mengenai kapasitas ponsel mereka.

### Negatif / Mitigasi
- Arsitektur manajemen model menjadi lebih modular (dimitigasi dengan kontrak `ModelCompatibilityChecker` yang bersih).
