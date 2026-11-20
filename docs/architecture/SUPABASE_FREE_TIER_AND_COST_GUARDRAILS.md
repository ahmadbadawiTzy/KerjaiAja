# PANDUAN GUARDRAILS SUPABASE FREE TIER & BIAYA RP 0 — KERJAIAJA
## Strategi Operasional Nol Rupiah Tanpa Risiko Over-Billing

**Dokumen:** `docs/architecture/SUPABASE_FREE_TIER_AND_COST_GUARDRAILS.md`  
**Status:** Wajib Dipatuhi (Strict Policy)  
**Target Anggaran:** Rp 0 / Bulan ($0 USD)

---

## 1. Batasan Kuota Supabase Free Tier

Supabase Free Plan menyediakan alokasi resource berikut:

| Komponen | Batas Free Tier | Alokasi Aman KerjaiAja (Guardrail) | Strategi Kepatuhan |
| :--- | :--- | :--- | :--- |
| **Database Space** | 500 MB | Maksimal 300 MB | Skema relasional ringkas, auto-purge log, vacuum |
| **File Storage** | 1 GB | Maksimal 700 MB | Batas upload 5MB, kompresi gambar, zero AI weights |
| **Bandwidth (Egress)**| 2 GB / bulan | Maksimal 1.2 GB / bulan | Offline-first sync diff, caching Room agresif |
| **Active Users (MAU)**| 50,000 MAU | 10,000 MAU (MVP) | Autentikasi opsional (Guest Mode mengurangi MAU) |
| **Realtime Messages** | 2,000,000 msg/bln | Maksimal 500,000 msg/bln | Filter CDC spesifik, disconnect saat background |
| **Realtime Concurrent**| 200 koneksi | Maksimal 100 koneksi | Unsubscribe channel saat screen exited |
| **Edge Functions** | 500,000 invokes | **0 invokes (TIDAK DIGUNAKAN)** | Logika inferensi berjalan 100% on-device |

---

## 2. Guardrail 1: Larangan Penyimpanan AI Weights di Supabase

- Model weights Google Gemma (1B / 270M) memiliki ukuran antara 350 MB hingga 1.2 GB per file.
- Mengunggah 1 file model AI ke Supabase Storage akan langsung menghabiskan 35%–100% kuota storage (1 GB) dan menguras egress bandwidth seketika hanya dalam 1–2 kali unduh.
- **Kebijakan Tegas:**
  - Bucket Supabase Storage hanya diizinkan untuk attachment tugas dokumen kecil (`task_attachments`).
  - Model weights Google Gemma **wajib** diunduh langsung dari kanal resmi Google AI Edge / Hugging Face public CDN yang gratis dan didesain untuk distribusi model skala besar.

---

## 3. Guardrail 2: Optimalisasi Database Space ($\le 500\text{ MB}$)

1. **Ukuran Record Rata-Rata:**
   - 1 entitas tugas (`Task`) dengan judul, deskripsi singkat, dan metadata membutuhkan ~250 byte di PostgreSQL.
   - 100,000 tugas aktif hanya memakan ~25 MB storage database.
2. **Pembersihan Rutin (Data Hygiene):**
   - Data soft-deleted (`deleted_at IS NOT NULL`) yang sudah berusia lebih dari 30 hari dibersihkan secara otomatis via scheduled vacuum atau cleanup script.
   - Tidak menyimpan riwayat log aktivitas pengguna secara rinci di database transactional.

---

## 4. Guardrail 3: Efisiensi Egress Bandwidth ($\le 2\text{ GB/bulan}$)

1. **Delta / Incremental Synchronization:**
   - Klien Android tidak pernah menarik seluruh database pengguna secara berulang (`SELECT * FROM tasks`).
   - Klien hanya meminta perubahan data yang dimutasi setelah timestamp sinkronisasi terakhir: `WHERE updated_at > :lastSyncTimestamp`.
2. **Local Caching di Room:**
   - Seluruh data bacaan dibaca dari database lokal Room (0 byte cloud egress).
   - Supabase hanya dihubungi saat terjadi mutasi lokal atau saat sinkronisasi periodik WorkManager.

---

## 5. Kebijakan Transparan Supabase Inactivity Pausing

- **Ketentuan Resmi Supabase Free Tier:** Proyek yang tidak menerima aktivitas API/kueri selama 7 hari berturut-turut akan memasuki status *Paused*.
- **Larangan Keras Skrip Anti-Pausing Manipulatif:**
  - **DILARANG** membuat atau menjalankan skrip terjadwal (seperti GitHub Actions cron job, cron bot, atau ping generator) yang sengaja melakukan kueri palsu hanya untuk mengelabui mekanisme *inactivity pause* Supabase.
  - Pendekatan manipulatif semacam ini melanggar *Fair Use Policy* dan prinsip integritas engineering KerjaiAja.
- **Penanganan Jujur & Transparan:**
  - Jika proyek testing/staging mengalami *pause* akibat masa libur atau tidak ada aktivitas pengguna, unpause dilakukan secara manual melalui Supabase Dashboard atau Supabase CLI resmi saat sesi development aktif kembali.
  - Untuk lingkungan produksi, aktivitas nyata siswa yang membuka aplikasi secara berkala sudah cukup menjaga proyek tetap aktif.

---

## 6. Guardrail 5: Larangan Mutlak Berkas Foto Soal di Supabase Storage

- **Privasi & Batas Kuota:** Berkas citra foto soal dari fitur Foto Soal & Pembahasan **TIDAK PERNAH** diunggah ke Supabase Storage.
- Pemrosesan citra bersifat 100% lokal (*on-device*) dan efemeral (file temporary langsung dihapus dari perangkat).
- Bucket Supabase Storage (kuota 1 GB) secara eksklusif hanya dialokasikan untuk lampiran dokumen tugas akademik resmi (PDF/dokumen kecil non-foto) yang secara sengaja diunggah oleh siswa untuk kolaborasi tugas.

---

## 7. Prosedur Darurat Bila Kuota Mendekati Limit (> 80%)

Jika penggunaan resource mencapai 80% dari batas Free Tier:
1. **Dilarang Menyalakan Billing Otomatis:** Akun Supabase harus tetap pada konfigurasi non-billing / spend cap $0.
2. **Nonaktifkan Realtime Sementara:** Fallback ke polling sinkronisasi manual jika kuota pesan Realtime menipis.
3. **Batasi Attachment Baru:** Turunkan batas unggah attachment dokumen dari 5 MB menjadi 1 MB.
4. **Data Purge:** Hapus soft-deleted records yang berumur lebih dari 7 hari.
