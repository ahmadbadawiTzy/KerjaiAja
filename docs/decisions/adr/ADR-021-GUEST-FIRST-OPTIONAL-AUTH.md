# ADR-021: Guest-First & Optional Cloud Authentication with Seamless Data Migration

## Status
Accepted

## Konteks
Sebagian besar pelajar dan mahasiswa menginginkan aplikasi produktivitas yang dapat langsung digunakan saat pertama kali diinstal tanpa hambatan registrasi paksa (onboarding friction). Memaksa pengguna membuat akun atau login di awal sering kali menyebabkan tingkat *drop-off* yang tinggi. Namun, jika pengguna ingin menyinkronkan data antar-perangkat atau bekerja dalam tim, autentikasi cloud tetap diperlukan.

## Keputusan
Kami mengadopsi pola **Guest-First & Optional Cloud Authentication**:
1. **Penggunaan Langsung Tanpa Akun (Guest Mode):**
   - Saat aplikasi pertama kali dibuka, pengguna langsung dapat membuat tugas, mata pelajaran, dan jadwal tanpa layar login yang memblokir.
   - Semua entitas data disimpan secara lokal di Room Database dengan identifier pengguna bertanda `is_guest = true` atau `user_id = local_guest_<uuid>`.
2. **Autentikasi Cloud Opsional (Supabase Auth):**
   - Pengguna dapat memilih untuk masuk (sign in) atau mendaftar (sign up) kapan saja melalui menu Profil atau saat mengakses fitur sinkronisasi cloud / kolaborasi tim.
   - Menggunakan Supabase Auth (`auth-kt`) via Email/Password atau Magic Link (Passwordless).
3. **Migrasi Data Transparan (Seamless Migration):**
   - Setelah pengguna berhasil login/signup untuk pertama kali, seluruh data lokal yang dibuat dalam Guest Mode secara otomatis dimigrasikan ke user ID Supabase mereka yang baru.
   - Antrean migrasi lokal dijalankan di background, memperbarui foreign key `user_id` di database lokal Room, dan menandai seluruh record untuk disinkronkan ke Supabase PostgreSQL.

## Konsekuensi
### Positif
- *Zero onboarding friction:* Pengguna dapat merasakan *core value* aplikasi dalam hitungan detik setelah instalasi.
- Privasi terlindungi bagi pengguna yang hanya menginginkan aplikasi offline lokal.
- Transisi yang mulus dari single-device guest menjadi cloud-synced account tanpa risiko kehilangan data tugas sebelumnya.

### Negatif / Mitigasi
- Kompleksitas transaksi database lokal saat migrasi `user_id` (dimitigasi dengan SQLite transaction atomic di Room).
- Penanganan potensi duplikasi jika akun cloud sudah memiliki data lama (dimitigasi dengan deteksi kesamaan UUID dan merge policy).
