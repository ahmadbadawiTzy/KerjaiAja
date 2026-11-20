# ADR-010: Explicit Opt-in Data Governance & Privacy-First Architecture

## Status
ACCEPTED (2026-11-20)

## Konteks
Aplikasi akademik mengelola data sensitif mahasiswa: nama lengkap, nomor induk mahasiswa (NIM), nama universitas/fakultas, jadwal kuliah, catatan tugas, nilai, dan lampiran dokumen tim. Penggunaan data pengguna untuk melatih model kecerdasan buatan (AI) tanpa persetujuan eksplisit merupakan pelanggaran hukum (UU Perlindungan Data Pribadi No. 27/2022 di Indonesia, GDPR di Uni Eropa) serta melanggar Google Play Store Developer Policy.

Diperlukan tata kelola data (*data governance*) yang transparan, dapat diaudit, dan memberikan kontrol penuh kepada pengguna (*privacy-by-design*).

## Keputusan
KerjaiAja menetapkan kebijakan **Explicit Opt-in Data Governance & Privacy Architecture**:
1. **Default Opt-Out untuk AI Training & Improvement:**
   - Secara default (saat registrasi awal), seluruh data pengguna **TIDAK PERNAH** digunakan untuk melatih atau meningkatkan model AI (fine-tuning dataset).
   - Pengguna harus secara sadar dan aktif mencentang toggle persetujuan (*explicit opt-in*) di menu Settings > AI & Privacy jika bersedia menyumbangkan data anonim untuk pengembangan model akademik.
2. **Data Sanitization & PII Redaction Pipeline:**
   - Bila pengguna melakukan opt-in, data yang masuk ke pipeline training wajib melalui proses pembersihan PII (Personally Identifiable Information) otomatis:
     - Nama mahasiswa, email, nomor telepon, NIM, URL eksternal, dan nama institusi spesifik disensor atau diganti dengan token anonim (misal: `[STUDENT_NAME]`, `[COURSE_NAME]`).
     - Deskripsi tugas yang terdeteksi mengandung kredensial, link gdrive privat, atau informasi rahasia otomatis dihapus dari dataset.
3. **Hak Menarik Persetujuan & Penghapusan Data (Right to Erasure):**
   - Pengguna berhak menarik persetujuan kapan saja (*revoke consent*).
   - KerjaiAja menyediakan fitur "Delete Account & All Associated Data" yang mengeksekusi penghapusan kaskade pada database Supabase (soft delete 30 hari grace period lalu hard purge permanen) sesuai standar Play Store.
4. **Isolasi Data Tim & Kontrol Akses Kolaboratif:**
   - Konten tugas kelompok dilindungi oleh PostgreSQL Row-Level Security (RLS). Hanya anggota tim yang berstatus aktif dalam project workspace yang dapat membaca atau memodifikasi tugas tersebut.

## Konsekuensi
### Positif
- Kepatuhan mutlak terhadap regulasi privasi global dan nasional (UU PDP, GDPR, Google Play Data Safety Section).
- Membangun rasa percaya tinggi (*user trust*) di kalangan mahasiswa dan institusi akademik.
- Mencegah risiko kebocoran data rahasia proyek tugas atau data pribadi ke bobot model AI.

### Negatif / Mitigasi
- Volume data latih riil dari pengguna mungkin lebih sedikit di awal.
  - *Mitigasi:* Menggunakan dataset sintetis terkurasi berbasis silabus dan studi kasus akademik terbuka untuk batch fine-tuning awal.
