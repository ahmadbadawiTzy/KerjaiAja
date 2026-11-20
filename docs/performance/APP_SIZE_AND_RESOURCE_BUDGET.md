# Anggaran Ukuran Aplikasi & Sumber Daya Perangkat (App Size & Resource Budget) — KerjaiAja
## Android Client Size · RAM Allocation · Supabase Client Overhead · On-Device Gemma

- **Tanggal Dokumen:** 2026-11-20
- **Status:** BASELINE LOCKED
- **Target Platform:** Android 10+ (API Level 29 – 35)
- **Baseline Ukuran:** Model AI Terpisah dari Paket APK/AAB

---

## 1. Prinsip Pemisahan Ukuran (Decoupled Size Architecture)

KerjaiAja memisahkan secara mutlak antara **binari aplikasi utama** dan **bobot model kecerdasan buatan (*AI model weights*)**:
1. **APK/AAB Ramping Secara Default:** Bobot model Gemma **TIDAK PERNAH DIBUNDEL** ke dalam APK atau Android App Bundle (AAB). Pengguna yang mengunduh KerjaiAja dari Google Play Store hanya mengunduh aplikasi produktivitas berukuran belasan megabyte (~18–20 MB).
2. **On-Demand Model Storage:** Bobot model (~250–950 MB) disimpan di direktori internal privat aplikasi (`context.noBackupFilesDir/models/`) hanya jika pengguna secara sadar memilih untuk mengunduhnya.
3. **Penyimpanan Terkelola & Hapus Instan:** Pengguna dapat menghapus bobot model sewaktu-waktu melalui menu Pengaturan untuk mengosongkan ruang disk ponsel tanpa kehilangan data tugas.
4. **Dilarang Menyimpan Model di Supabase Storage:** Model AI wajib diunduh dari CDN resmi Google AI Edge / Hugging Face untuk menjaga kuota 1 GB Supabase Free Tier.

---

## 2. Anggaran Ukuran Binari Aplikasi (Binary Size Budget)

| Komponen Paket | Target Anggaran (AAB) | Target Anggaran (APK Universal) | Estimasi Aktual R8 Minified |
| :--- | :---: | :---: | :---: |
| **Android Core & UI (Compose M3)** | $\le 5.0\text{ MB}$ | $\le 7.0\text{ MB}$ | ~4.2 MB |
| **Room Database & SQLite FTS5** | $\le 2.0\text{ MB}$ | $\le 2.5\text{ MB}$ | ~1.6 MB |
| **Supabase Kotlin Client & Ktor 3.x** | $\le 3.5\text{ MB}$ | $\le 4.5\text{ MB}$ | ~3.0 MB |
| **LiteRT-LM C++ Runtime (.so)** | $\le 6.0\text{ MB}$ | $\le 8.5\text{ MB}$ | ~5.8 MB (per ABI arm64-v8a) |
| **Kotlinx Serialization & Coroutines**| $\le 1.5\text{ MB}$ | $\le 2.0\text{ MB}$ | ~1.1 MB |
| **Aset Visual & Font Inter** | $\le 1.5\text{ MB}$ | $\le 1.8\text{ MB}$ | ~1.2 MB |
| **Total Ukuran Aplikasi Bersih** | **$\le 19.5\text{ MB}$** | **$\le 26.3\text{ MB}$** | **~16.9 MB (AAB download)** |

---

## 3. Anggaran Memori RAM & Batasan LMKD (RAM Budget)

Sistem Android memantau alokasi RAM secara ketat via Low Memory Killer Daemon (LMKD).

| Komponen Sistem | Mode A: Idle (Tanpa AI) | Mode B: Gemma 270M (`LIMITED`) | Mode B: Gemma 3 1B (`RECOMMENDED`) |
| :--- | :---: | :---: | :---: |
| **Android OS Base Overhead** | ~40 MB | ~40 MB | ~40 MB |
| **Jetpack Compose UI Hierarchy** | ~35 MB | ~45 MB | ~45 MB |
| **Room Database & Sync Queue** | ~15 MB | ~20 MB | ~25 MB |
| **Supabase Client (Ktor Engine)**| ~12 MB | ~12 MB | ~15 MB |
| **LiteRT-LM Runtime & KV-Cache** | 0 MB *(Unloaded)* | ~350 MB | ~850 – 1.050 MB |
| **Total Peak Memory Usage** | **~102 MB** | **~467 MB** | **~1.175 MB** |
| **Toleransi Minimum RAM Fisik** | **2 GB RAM** | **4 GB RAM** | **6 GB RAM** |

### Aturan Lifecycle Manajemen Memori:
- Model LiteRT-LM di-load secara on-demand saat sesi Asisten Kerja aktif.
- Auto-unload model dari RAM setelah 5 menit tidak ada interaksi.
- Merespons `ComponentCallbacks2.onTrimMemory(level)`: Jika level mencapai `TRIM_MEMORY_RUNNING_CRITICAL`, runtime model AI segera dilepaskan dari RAM.

---

## 4. Anggaran Daya Baterai & Suhu Termal

| Parameter Performa | Target Maksimum | Strategi Rekayasa |
| :--- | :---: | :--- |
| **Konsumsi Daya Per Inferensi** | $\le 0.05\%$ baterai per aksi | Context window dibatasi $\le 1024$ token dan durasi singkat ($1.5–3.5$ detik). |
| **Pemicu Inferensi Otomatis** | **0 kali (Zero Auto-Run)** | AI hanya aktif atas perintah eksplisit pengguna. Dilarang menjalankan inferensi AI di background worker. |
| **Thermal Throttling Protection** | Deteksi `PowerManager.getThermalStatus()` | Jika suhu perangkat mencapai `THERMAL_STATUS_SEVERE`, aksi AI ditangguhkan sementara. |

---

## 5. Anggaran Performa Startup & Responsivitas (Performance SLA)

| Metrik Performa | Target SLA | Metode Verifikasi |
| :--- | :---: | :--- |
| **Cold Startup Time (Klik Ikon -> Interaktif)**| **$\le 800\text{ ms}$** | Baseline Profiles, lazy injection Supabase/LiteRT. |
| **Warm Startup Time** | $\le 300\text{ ms}$ | State restoration dari saved instance state. |
| **Frame Rate Rendering UI** | **60 FPS stabil (16.6ms/frame)** | Compose Layout Inspector, zero heavy compute di UI thread. |
| **Kueri SQLite FTS5 (BM25 Top-5)** | **$\le 25\text{ ms}$** | Indeks teroptimasi pada `tasks_fts` dengan pembatasan kueri `LIMIT 5`. |
| **Deterministic Priority Score Calculation** | **$\le 5\text{ ms}$ per 100 tugas** | Komputasi CPU murni berbasis formula matematika instan. |
