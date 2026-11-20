# ADR-038: Shared Database Schema and Multi-Tenant Row Level Security for CMS
## Integrasi Skema Basis Data Bersama Android-CMS dan Kompatibilitas Delta Sync

- **Status:** APPROVED / LOCKED
- **Tanggal:** 2026-11-20
- **Penanggung Jawab:** Database Architect, Senior Android Engineer, Senior Full-Stack Web Engineer
- **Konteks:** Menjaga kesatuan sumber data kebenaran antara aplikasi mobile Android dan Web Admin CMS di basis data Supabase PostgreSQL tanpa merusak alur sinkronisasi offline.

---

## 1. Konteks & Masalah

Pengembangan Web Admin CMS seringkali memicu godaan untuk membuat basis data terpisah atau menambahkan tabel baru tanpa mempertimbangkan dampaknya terhadap klien mobile offline-first.

Tantangan arsitektur yang dihadapi:
1. Mencegah pembuatan basis data kedua yang menduplikasi data dan melipatgandakan biaya operasional.
2. Memastikan mutasi data referensi akademik di CMS dapat diterima oleh klien Android yang sedang offline saat perangkat kembali terhubung ke internet.
3. Melarang CMS memanipulasi basis data lokal Room Android secara langsung.
4. Menjaga batas ukuran basis data PostgreSQL di bawah kuota aman Supabase Free Tier ($\le 500\text{ MB}$).

---

## 2. Keputusan Arsitektur

Kami memutuskan untuk mengintegrasikan skema basis data bersama dengan aturan rekayasa berikut:

1. **Satu Basis Data PostgreSQL Terpadu (Single Shared Instance):**
   - Android dan Web CMS terhubung ke proyek Supabase PostgreSQL yang sama.
   - Hak akses dipisahkan murni melalui kebijakan RLS (Row Level Security).
2. **Kepatuhan Terhadap Sinkronisasi Offline Android (Delta Sync Preservation):**
   - Tabel `public.academic_references` dan `public.curriculum_topics` memiliki kolom `updated_at TIMESTAMPTZ` yang diperbarui otomatis via trigger setiap kali admin melakukan mutasi data di CMS.
   - Klien Android menarik perubahan baru menggunakan kueri Delta Sync:
     `SELECT * FROM academic_references WHERE updated_at > :lastSyncTime`.
3. **Pola Penghapusan Lembut (Soft-Deletion Tombstone):**
   - Penghapusan referensi kurikulum oleh admin menggunakan status `is_published = false` atau pengisian kolom `deleted_at`, sehingga klien Android dapat menonaktifkan data lokal tanpa terjadi anomali data yatim (*orphan data*).
4. **Isolasi Alur Data Umpan Balik (`public.user_feedbacks`):**
   - Siswa Android dapat menyisipkan (*INSERT*) tiket keluhan atau laporan OCR.
   - Staf Admin dan Support dapat membaca (*SELECT*) dan memperbarui status tiket (*UPDATE* status & catatan resolusi).
   - Pengguna Android hanya dapat membaca tiket milik mereka sendiri (`auth.uid() = user_id`).
5. **Guardrails Ukuran Basis Data & Retensi:**
   - Log audit administratif diatur dengan kebijakan retensi 90 hari untuk log biasa, menjaga konsumsi penyimpanan PostgreSQL tetap di bawah 50 MB pada skala MVP.

---

## 3. Konsekuensi

### Positif:
- **Biaya Infrastruktur Rp0:** Tidak ada penambahan biaya server database terpisah; tetap berada dalam kuota aman Supabase Free Tier.
- **Konsistensi Data Real-Time:** Perubahan materi kurikulum langsung tersedia secara konsisten bagi seluruh ekosistem tanpa rilis ulang APK.
- **Integritas Offline Android Terjamin:** Tidak ada gangguan pada performa baca/tulis instan di Room Database lokal Android.

### Negatif / Mitigasi:
- **Ketergantungan Pada Disiplin RLS:** Kesalahan konfigurasi policy RLS berpotensi mengekspos data antar-klien. Mitigasi: Test suite otomatis `CMS_TEST_STRATEGY.md` menyertakan pengujian RLS wajib yang dijalankan di CI sebelum migrasi diterapkan ke produksi.

---

## 4. Referensi Dokumen Terkait

- [CMS Data Architecture](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/CMS_DATA_ARCHITECTURE.md)
- [Supabase Architecture](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/SUPABASE_ARCHITECTURE.md)
- [Sync Architecture](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/SYNC_ARCHITECTURE.md)
- [Supabase Free Tier Guardrails](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/SUPABASE_FREE_TIER_AND_COST_GUARDRAILS.md)
