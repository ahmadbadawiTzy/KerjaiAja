# ADR-001: Deteksi Kapabilitas Perangkat Adaptif untuk Local AI (Local AI Capability Detection)

> [!NOTE]
> **Status:** **CONSOLIDATED INTO ADR-024 & ADR-035** (Rancangan Awal Pra-Rebaseline)  
> Dokumen ini merupakan rancangan awal deteksi kapabilitas perangkat untuk local AI. Seluruh keputusan dan mekanismenya telah dilebur dan disempurnakan ke dalam:
> - **[ADR-024: Device Capability Management](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-024-DEVICE-CAPABILITY-MANAGEMENT.md)**
> - **[ADR-035: Per-Model and Per-Capability Device Evaluation Gate](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-035-PER-MODEL-DEVICE-CAPABILITY-EVALUATION.md)**  
> Nomor kanonikal ADR-001 pada master index dialokasikan untuk **[ADR-001: Android Native, Kotlin & Jetpack Compose](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-001-ANDROID-NATIVE-JETPACK-COMPOSE.md)** (Status: ACCEPTED).

- **Status:** CONSOLIDATED INTO ADR-024 & ADR-035
- **Tanggal:** 2026-11-20
- **Pengambil Keputusan:** Tim Arsitektur KerjaiAja (Disetujui Pengguna)
- **Terkait Dokumen:** [`docs/ai/LOCAL_AI_ARCHITECTURE.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/ai/LOCAL_AI_ARCHITECTURE.md), [`docs/decisions/adr/ADR-024-DEVICE-CAPABILITY-MANAGEMENT.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-024-DEVICE-CAPABILITY-MANAGEMENT.md), [`docs/decisions/adr/ADR-035-PER-MODEL-DEVICE-CAPABILITY-EVALUATION.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-035-PER-MODEL-DEVICE-CAPABILITY-EVALUATION.md)

---

## 1. Konteks (Context)

Aplikasi KerjaiAja mengusung fitur on-device LLM (Gemma 3 1B IT via LiteRT-LM) untuk membantu dekonstruksi tugas akademik (*assignment breakdown*) secara privat dan offline. Sebelumnya, dokumen spesifikasi mencantumkan target perangkat dengan batasan sederhana "RAM 4–6 GB, API 29+".

Audit rekayasa mendalam (`docs/audits/SUPERPOWERS_DOCUMENTATION_AUDIT.md` temuan AUD-CRIT-01) menunjukkan bahwa pada perangkat Android dengan total RAM fisik 4 GB, memori bebas yang tersedia (*available RAM*) saat sistem idle sering kali hanya berkisar 800–1.200 MB karena sebagian besar memori dikonsumsi oleh OS Android, System UI, Google Mobile Services (GMS), dan background services vendor (OEM skin). Lonjakan memori inferensi hingga ~1.2 GB pada perangkat 4 GB hampir pasti akan memicu pemutusan paksa proses oleh Linux/Android *Low Memory Killer Daemon* (LMKD).

## 2. Masalah (Problem)

Menjadikan kapasitas RAM fisik sebagai satu-satunya indikator kelayakan dapat menyebabkan *false positive* (mengizinkan eksekusi pada ponsel 4 GB yang berujung pada crash mendadak) atau *false negative* (menolak ponsel dengan RAM cukup namun menolak optimasi adaptif). Selain itu, performa inferensi pada chipset kelas bawah dapat menyebabkan *thermal throttling* ekstrim dan penurunan daya baterai yang drastis.

## 3. Keputusan (Decision)

Mengadopsi **Adaptive Multi-Factor Capability Detection** dengan klasifikasi tiering perangkat:

1. **Tier Rendah (RAM 4 GB):** Menggunakan **mode fallback manual sebagai default**. Model on-device tidak diaktifkan secara otomatis untuk mencegah risiko crash LMK.
2. **Tier Menengah (RAM 6 GB):** Ditetapkan sebagai **kandidat pengujian (*test candidate*)** Local AI. Aktivasi memerlukan lolos evaluasi multi-faktor saat runtime.
3. **Tier Atas (RAM >= 8 GB):** Ditetapkan sebagai **kandidat prioritas pengujian** dengan alokasi context window optimal.
4. **Evaluasi Runtime Multi-Faktor:** Aktivasi AI tidak hanya memeriksa total RAM, melainkan mengevaluasi parameter dinamis sebelum inisialisasi:
   - **Available Memory:** Memori bebas aktual (`ActivityManager.MemoryInfo.availMem`) saat runtime wajib mencukupi ambang aman (target awal: minimal 1.5 GB bebas).
   - **Memory Pressure:** Memonitor sinyal sistem via `ComponentCallbacks2.onTrimMemory()`.
   - **Storage Availability:** Ruang penyimpanan internal bebas minimal 2.5 GB (untuk unduhan bobot model dan ruang kerja buffer).
   - **ABI & Runtime:** Arsitektur 64-bit (`arm64-v8a`) dan kompatibilitas binary LiteRT-LM.
   - **Thermal & Battery Status:** Baterai > 20% (atau terhubung ke charger) dan status suhu perangkat tidak dalam kondisi `THERMAL_STATUS_SEVERE`.
5. **Graceful Fallback:** Jika salah satu syarat tidak terpenuhi, aplikasi secara otomatis menampilkan antarmuka manual konvensional tanpa mengganggu alur produktivitas pengguna.

## 4. Alternatif yang Dipertimbangkan (Alternatives Considered)

- **Alternatif A (Threshold RAM Tunggal 4 GB):** Ditolak karena risiko crash OOM/LMKD sangat tinggi di ekosistem Android nyata.
- **Alternatif B (Menghapus Fitur AI Sepenuhnya):** Ditolak karena on-device AI merupakan diferensiasi utama KerjaiAja.
- **Alternatif C (Mengharuskan RAM >= 6 GB secara Kaku):** Dipertimbangkan, namun diputuskan lebih fleksibel dengan menjadikannya tier pengujian adaptif yang mengevaluasi memori bebas riil.

## 5. Konsekuensi (Consequences)

### Positif:
- Menghilangkan risiko crash mendadak akibat LMK pada perangkat pengguna kelas menengah-bawah.
- Aplikasi tetap deterministik dan 100% fungsional untuk seluruh pengguna melalui fallback manual.
- Menyediakan jalur pengujian terkontrol untuk perangkat RAM 6 GB dan 8 GB+.

### Negatif:
- Pengguna ponsel RAM 4 GB tidak dapat menikmati fitur asistensi AI otomatis secara lokal (namun tetap mendapatkan fungsionalitas manajemen tugas penuh).
- Menambah kompleksitas logika inspeksi perangkat pada modul data/ai (`DeviceCapabilityDetector`).

## 6. Risiko & Mitigasi (Risks & Mitigations)

- **Risiko:** Fragmentasi vendor Android (OEM agresif seperti Xiaomi/Samsung yang menghentikan background worker).
- **Mitigasi:** Pengecekan status memori dilakukan sesaat sebelum sesi inferensi dimulai (*just-in-time check*).

## 7. Persyaratan Validasi (Validation Requirements)

- `REQUIRES TECHNICAL VALIDATION`: Pengujian profil memori riil menggunakan Android Profiler pada minimal 3 perangkat fisik berbeda (RAM 4 GB, 6 GB, dan 8 GB) untuk memvalidasi ambang batas `availMem` yang aman.
