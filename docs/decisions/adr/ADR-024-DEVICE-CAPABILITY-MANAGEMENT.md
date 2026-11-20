# ADR-024: Device Capability Management & Multi-Factor Hardware Qualification for On-Device AI

## Status
Accepted

## Konteks
Menjalankan Large Language Model on-device seperti Google Gemma (1B parameter atau 270M) membutuhkan konsumsi memori RAM yang signifikan (~600 MB hingga 1.2 GB saat aktif), instruksi komputasi intensif (NEON/GPU), dan ruang penyimpanan internal (~500 MB – 1.5 GB). Jika model dijalankan secara gegabah pada perangkat yang kekurangan memori atau mengalami panas ekstrem (thermal throttling), sistem operasi Android dapat melakukan *Out-Of-Memory (OOM) kill* terhadap aplikasi, menyebabkan crash mendadak, lag antarmuka, dan pengurasan baterai yang cepat.

## Keputusan
Kami mengimplementasikan modul terpusat **`DeviceCapabilityManager`** yang melakukan evaluasi hardware multi-faktor sebelum mengizinkan pengunduhan atau eksekusi model on-device:

### 1. Faktor-Faktor Evaluasi Hardware
1. **Total & Available RAM:** Memeriksa `ActivityManager.MemoryInfo`. Wajib memiliki RAM bebas aktual minimal **1.5 GB** untuk Gemma 1B, atau **800 MB** untuk Gemma 270M.
2. **Arsitektur CPU (ABI):** Wajib mendukung 64-bit ABI (`arm64-v8a` atau `x86_64`). Perangkat 32-bit (`armeabi-v7a`) otomatis dinyatakan tidak didukung.
3. **Penyimpanan Internal Bebas (Storage):** Memeriksa `StatFs` direktori privat aplikasi. Wajib memiliki ruang kosong minimal **3.0 GB** sebelum mengunduh model.
4. **Status Termal (Thermal Throttling):** Menggunakan `PowerManager.getThermalStatus()`. Jika status berada pada level `THERMAL_STATUS_SEVERE` atau `CRITICAL`, inferensi AI diblokir sementara.
5. **Kondisi Daya / Baterai:** Memeriksa level baterai ($\ge 30\%$) dan status pengisian daya (`BatteryManager`).

### 2. Tingkatan Kualifikasi Perangkat (Qualification Tiers)
- **`RECOMMENDED` (Direkomendasikan):** RAM $\ge 6\text{ GB}$, RAM bebas $\ge 2.5\text{ GB}$, storage bebas $\ge 5\text{ GB}$, 64-bit CPU. Model Gemma 1B IT berjalan penuh dengan akselerasi GPU/NPU.
- **`LIMITED` (Terbatas / Ringan):** RAM $4\text{ GB} - 6\text{ GB}$, RAM bebas $1.2\text{ GB} - 2.5\text{ GB}$, storage bebas $\ge 3\text{ GB}$. Direkomendasikan Gemma 270M atau Gemma 1B quant Q4 CPU-only dengan konteks terbatas (512 token).
- **`NOT_RECOMMENDED` (Kurang Disarankan):** RAM $< 4\text{ GB}$ atau RAM bebas $< 1.0\text{ GB}$. Pengguna diberi peringatan risiko performa/baterai, model tidak diaktifkan secara default.
- **`UNSUPPORTED` (Tidak Didukung):** 32-bit ABI, storage $< 2\text{ GB}$, atau OS Android di bawah Android 10 (API 29). Opsi unduh model dinonaktifkan sepenuhnya.

## Konsekuensi
### Positif
- Mencegah crash OOM aplikasi dan menjaga stabilitas sistem perangkat pengguna.
- Transparansi penuh kepada pengguna mengenai kondisi perangkat mereka.
- Menghemat kuota internet pengguna dengan mencegah pengunduhan model yang tidak dapat dijalankan.

### Negatif / Mitigasi
- Sebagian pengguna tidak dapat menikmati fitur on-device AI (dimitigasi dengan Mode A yang tetap 100% lengkap dan fungsional).
