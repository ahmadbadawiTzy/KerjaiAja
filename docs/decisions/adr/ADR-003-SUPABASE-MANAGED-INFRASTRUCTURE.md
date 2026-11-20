# ADR-003: Supabase as Managed Infrastructure

- **Status:** ACCEPTED
- **Tanggal:** 2026-11-20
- **Pengambil Keputusan:** Tim Arsitektur KerjaiAja (Senior Platform Architect & Backend Engineer)
- **Terkait Dokumen:** [`docs/architecture/SYSTEM_ARCHITECTURE.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/SYSTEM_ARCHITECTURE.md), [`docs/architecture/DATA_ARCHITECTURE.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/DATA_ARCHITECTURE.md)

---

## 1. Konteks (Context)
KerjaiAja memerlukan infrastruktur basis data relasional yang andal, mekanisme otentikasi pengguna yang aman, kemampuan realtime untuk kolaborasi tim, dan penyimpanan objek (lampiran berkas tugas) tanpa perlu mengelola server database PostgreSQL mentah sendiri dari nol.

## 2. Keputusan (Decision)
Memanfaatkan **Supabase** secara eksklusif sebagai **Managed Infrastructure**, dengan aturan rekayasa tegas:
1. **Pemisahan Peran Arsitektur:**
   - **Go Backend API:** Pintu gerbang utama seluruh logika bisnis, validasi izin, perhitungan prioritas, dan orkestrasi AI.
   - **Supabase Auth:** Mengelola pendaftaran akun, login, hashing kata sandi, dan penerbitan JSON Web Token (JWT).
   - **Supabase PostgreSQL:** Menyimpan data relasional (`users`, `tasks`, `teams`, `team_members`, `activity_logs`).
   - **Supabase Realtime:** Digunakan untuk mendengarkan perubahan status tugas tim secara efisien.
   - **Supabase Storage:** Menyimpan berkas lampiran tugas akademik (PDF, DOCX, ZIP).
2. **Kerahasiaan Kredensial:**
   - Klien Android **TIDAK PERNAH** memegang `service_role_key`. Klien Android hanya memegang Supabase `anon_key` untuk Auth SDK dan berkomunikasi dengan Go Backend melalui Bearer JWT token.
   - Go Backend memverifikasi tanda tangan JWT Supabase menggunakan public key atau secret key melalui middleware.
3. **Defense-in-Depth RLS:**
   - PostgreSQL Row-Level Security (RLS) tetap dirancang dan aktif sebagai lapisan pertahanan ganda, namun tidak menggantikan validasi otorisasi di dalam kode Go Backend.

## 3. Alternatif yang Dipertimbangkan (Alternatives Considered)
- **Self-Hosted PostgreSQL + Docker:** Sangat baik untuk dev lokal, namun Supabase Free Tier dipilih karena menyediakan bundel lengkap (Auth, Realtime, Storage, Migration tools) secara gratis ($0) tanpa overhead konfigurasi manual.
- **Firebase:** Ditolak karena struktur NoSQL Firestore menyulitkan kueri relasional berjenjang (team tasks, filtering prioritas, audit log) dan vendor lock-in yang tinggi.

## 4. Konsekuensi (Consequences)
- **Positif:** Menghemat waktu penyusunan modul autentikasi dan storage hingga 70%; performa database PostgreSQL kelas enterprise; migrasi skema terstandarisasi.
- **Negatif:** Adanya ketergantungan pada format JWT dan interface Supabase.
- **Implikasi Keamanan:** Proteksi ketat terhadap secret; RLS mencegah kebocoran data jika terjadi kesalahan query di client; validasi hak akses tim selalu dievaluasi di Go middleware.
- **Implikasi Biaya:** 100% Gratis menggunakan Supabase Free Tier ($0/bulan) untuk pengembangan dan rilis awal.
