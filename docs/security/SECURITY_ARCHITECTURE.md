# ARSITEKTUR KEAMANAN & PRIVASI (SECURITY ARCHITECTURE) — KERJAIAJA
## Secure-by-Design · Zero Secret Exposure · Row Level Security · Privacy-First

**Dokumen:** `docs/security/SECURITY_ARCHITECTURE.md`  
**Status:** Baseline Disetujui  
**Kepatuhan:** OWASP MASVS, Google Play Store Security Guidelines, GDPR / UU PDP

---

## 1. Model Ancaman & Prinsip Keamanan

KerjaiAja menerapkan prinsip **Zero-Trust Client** dan **Least Privilege**:
1. Aplikasi mobile Android berjalan di lingkungan pengguna yang tidak tepercaya (berpotensi di-decompile atau di-inspect lalu lintas jaringannya).
2. Kunci Supabase Service Role (`service_role_key`) **DILARANG KERAS** disimpan di APK, repository, atau environment variable aplikasi Android.
3. Klien hanya dibekali kunci publik anonim (`anon_key`), yang fungsinya terbatas pada gerbang masuk PostgREST dengan pengamanan mutlak di level PostgreSQL Row Level Security (RLS).
4. Data AI pengguna 100% diproses secara lokal di perangkat dan tidak pernah meninggalkan sandbox aplikasi.

---

## 2. Manajemen Kunci & Kredensial

### A. Kunci Publik Anonim (`anon_key`) vs Kunci Layanan (`service_role_key`)
- **`SUPABASE_ANON_KEY`:** Kunci publik yang aman untuk disertakan dalam binary aplikasi Android (via `BuildConfig` yang diambil dari `local.properties`). Kunci ini merepresentasikan entitas anonim atau pengguna terautentikasi (melalui JWT header `Authorization: Bearer <token>`).
- **`SUPABASE_SERVICE_ROLE_KEY`:** Kunci administratif bypass RLS. **TIDAK PERNAH** dimasukkan ke dalam project Android atau repository Git. Hanya ada di dasbor Supabase untuk keperluan devops backend darurat.

### B. Penyimpanan Token Sesi di Perangkat
- Refresh token dan access token JWT Supabase disimpan secara aman di perangkat menggunakan pustaka AndroidX `EncryptedSharedPreferences` / Jetpack DataStore yang dienkripsi oleh **Android Keystore System** dengan algoritma AES-256-GCM.
- Token tidak pernah disimpan dalam plain-text SQLite atau shared preferences biasa.

---

## 3. Penegakan PostgreSQL Row Level Security (RLS)

Seluruh tabel publik di Supabase PostgreSQL diproteksi dengan RLS aktif:
- Semua kueri `SELECT`, `INSERT`, `UPDATE`, `DELETE` secara otomatis menyaring baris berdasarkan klausa `auth.uid()`.
- Pengguna A tidak dapat membaca atau memodifikasi tugas Pengguna B, meskipun Pengguna A memanggil REST endpoint PostgREST secara manual dengan UUID Pengguna B.
- Untuk kolaborasi tim: Hak akses diverifikasi silang dengan tabel relasi `team_members`. Pengguna yang bukan anggota tim otomatis menerima error `403 Forbidden` / `404 Not Found` dari database.

---

## 4. Keamanan & Privasi On-Device AI

1. **Zero External AI Traffic:**
   - Tidak ada panggilan HTTP ke Gemini API, OpenAI API, Anthropic, atau server AI eksternal apa pun.
   - Analisis audit jaringan (network sniffer) dapat membuktikan bahwa kueri Asisten Kerja tidak menghasilkan payload keluar.
2. **Penyimpanan Bobot Model di Private App Sandbox:**
   - Model weights Google Gemma disimpan di direktori privat aplikasi:
     `context.noBackupFilesDir.absolutePath + "/models/"`
   - File model memiliki flag permissions `0600` (hanya dapat dibaca oleh UID aplikasi KerjaiAja).
   - Penggunaan `noBackupFilesDir` memastikan bobot model AI tidak ikut terunggah ke Google Cloud Backup otomatis pengguna (menghemat kuota Google Drive pengguna).
3. **Pembersihan Konteks Memori (RAM Sanitization):**
   - Context buffer teks tugas di-reset segera setelah inferensi selesai untuk mencegah residu data di memori heap aplikasi.

---

## 5. Kepatuhan Play Store & Regulasi Perlindungan Data

1. **Fitur Penghapusan Akun Lengkap (Account Deletion Requirement):**
   - Sesuai kebijakan Google Play Store 2024–2026, aplikasi menyediakan tombol *"Hapus Akun & Seluruh Data"* di menu Pengaturan Akun.
   - Menghapus akun akan memicu fungsi RPC PostgreSQL:
     ```sql
     DELETE FROM auth.users WHERE id = auth.uid();
     ```
     Dengan klausa `ON DELETE CASCADE`, seluruh profil, tugas, sub-tugas, keanggotaan tim, dan file attachment milik pengguna otomatis terhapus bersih dari database cloud.
2. **Izin Aplikasi Minimal (Minimal Permissions):**
   - Aplikasi hanya meminta izin `INTERNET` dan `ACCESS_NETWORK_STATE`.
   - Tidak meminta izin lokasi, kontak, SMS, atau kamera.
   - Model download menggunakan download manager internal dengan penyimpanan privat (tanpa membutuhkan izin `WRITE_EXTERNAL_STORAGE`).
