# ADR-035: Per-Model and Per-Capability Device Evaluation Gate

## Status
ACCEPTED (Disetujui) — 2026-11-20

## Konteks
Pendekatan kualifikasi perangkat keras konvensional sering kali memberikan satu label kaku (*monolithic device tier*) untuk keseluruhan aplikasi (misalnya menandai perangkat RAM 4 GB sebagai "UNSUPPORTED" untuk seluruh fitur AI). Pada kenyataannya, perangkat kelas menengah tersebut sangat mampu menjalankan engine OCR lokal berbobot 30 MB dan model bahasa ultra-ringan Google Gemma 3 270M secara mulus, meskipun tidak mampu menjalankan model visual multimodal 2 GB. 

Menolak seluruh fitur AI pada perangkat kelas menengah membatasi inklusivitas aplikasi bagi pelajar Indonesia. Sebaliknya, mengizinkan model besar berjalan di perangkat terbatas akan memicu *force close*.

Diperlukan evaluasi hardware yang terarah dan granular per-model dan per-kemampuan (*per-model qualification gate*).

## Keputusan
Menetapkan kebijakan evaluasi kapabilitas hardware terarah per-model dan per-fitur pada komponen `DeviceCapabilityManager`:
1. **Pemisahan 4 Gerbang Kapabilitas Independen:**
   - *Gerbang OCR (Google ML Kit):* RAM total $\ge 2.0\text{ GB}$, RAM bebas $\ge 300\text{ MB}$, Android 8.0+ (Universal untuk hampir 100% ponsel).
   - *Gerbang Model Teks Ringan (Gemma 3 270M):* RAM total $\ge 3.8\text{ GB}$, RAM bebas $\ge 1.2\text{ GB}$, storage $\ge 2.5\text{ GB}$, ABI 64-bit.
   - *Gerbang Model Teks Lengkap (Gemma 3 1B IT):* RAM total $\ge 6.0\text{ GB}$, RAM bebas $\ge 2.5\text{ GB}$, storage $\ge 4.0\text{ GB}$, ABI 64-bit, akselerasi GPU/NPU.
   - *Gerbang Model Visual Multimodal (Gemma 3n E2B Lab):* RAM total $\ge 8.0\text{ GB}$, RAM bebas $\ge 3.0\text{ GB}$, storage $\ge 5.0\text{ GB}$, NPU $\ge 15\text{ TOPS}$.
2. **Transparansi UX Pengguna:**
   - Dialog evaluasi hardware menampilkan rincian spesifikasi fisik (RAM terpasang, RAM bebas, sisa storage).
   - Menampilkan estimasi ukuran berkas unduhan dan kebutuhan disk setelah ekstraksi secara jujur.
   - **Dilarang keras melakukan auto-download model di latar belakang:** Pengguna wajib memberikan persetujuan eksplisit (*opt-in*).
3. **Integritas Unduhan & Manajemen Penyimpanan:**
   - Unduhan mendukung pemulihan otomatis (*resumable download*) via Android `DownloadManager`.
   - Integritas berkas divalidasi menggunakan verifikasi checksum SHA-256 terhadap manifest resmi sebelum model diaktifkan.
   - Disediakan tombol *"Hapus Model"* satu ketukan di menu Pengaturan untuk membebaskan ruang penyimpanan seketika.

## Konsekuensi
- **Positif:**
  - Inklusivitas maksimal: Pelajar dengan ponsel RAM 4 GB tetap dapat menikmati OCR dan Asisten Teks Ringan 270M.
  - Perlindungan perangkat: Mencegah ponsel mid-range memaksakan eksekusi model multimodal yang berpotensi crash.
  - Kedaulatan pengguna penuh atas kuota data dan penyimpanan internal mereka.
- **Negatif / Tantangan:**
  - Diperlukan pengelolaan beberapa profil model bobot di CDN eksternal dan pemeliharaan manifest SHA-256 yang sinkron.

## Dokumen Terkait
- [DEVICE_CAPABILITY_MANAGER.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/DEVICE_CAPABILITY_MANAGER.md)
- [DEVICE_CAPABILITY_STRATEGY.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/ai/DEVICE_CAPABILITY_STRATEGY.md)
- [ADR-024-DEVICE-CAPABILITY-MANAGEMENT.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-024-DEVICE-CAPABILITY-MANAGEMENT.md)
- [ADR-029-SEPARATION-OF-TEXT-AND-VISION-CAPABILITY.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-029-SEPARATION-OF-TEXT-AND-VISION-CAPABILITY.md)
- [ADR-030-MULTIMODAL-VISION-MODEL-EVALUATION-STRATEGY.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-030-MULTIMODAL-VISION-MODEL-EVALUATION-STRATEGY.md)
