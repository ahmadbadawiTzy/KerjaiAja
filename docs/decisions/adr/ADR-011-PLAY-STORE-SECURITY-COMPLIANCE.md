# ADR-011: Google Play Store Compliance & Security by Design

## Status
ACCEPTED (2026-11-20)

## Konteks
Google Play Store memberlakukan standar keamanan dan privasi yang sangat ketat bagi aplikasi Android yang didistribusikan ke publik, antara lain:
- Target SDK versi modern (saat ini Target SDK 34/35 wajib untuk rilis baru).
- Deklarasi formulir Google Play Data Safety yang jujur dan dapat diverifikasi.
- Penyediaan alur penghapusan akun (*account deletion flow*) yang dapat diakses dari dalam aplikasi maupun melalui URL publik eksternal.
- Prinsip *Permission Minimization* (larangan meminta izin sensitif yang tidak relevan dengan fungsionalitas inti aplikasi).
- Kebijakan konten AI generatif (penyediaan mekanisme pelaporan/flagging konten yang tidak pantas).

Ketidakpatuhan terhadap regulasi ini dapat mengakibatkan penolakan saat review (*rejection*) atau penghapusan aplikasi (*suspension*) dari Google Play Store.

## Keputusan
KerjaiAja menerapkan prinsip **Play Store Compliance & Security by Design** sejak fase inisiasi proyek:
1. **Target SDK & Toolchain Android Modern:**
   - `compileSdk = 35`, `targetSdk = 35`, `minSdk = 26` (Android 8.0 Oreo, mencakup >95% perangkat aktif Android global).
   - Menggunakan Android App Bundle (`.aab`) dan Google Play App Signing.
2. **Prinsip Minimisasi Izin (Permission Minimization):**
   - Hanya meminta izin yang mutlak dibutuhkan:
     - `android.permission.INTERNET` (akses API backend Go & Supabase).
     - `android.permission.POST_NOTIFICATIONS` (Android 13+, untuk reminder tugas akademik dan update tim dengan persetujuan eksplisit).
     - `android.permission.SCHEDULE_EXACT_ALARM` / `USE_EXACT_ALARM` (hanya jika diperlukan untuk alarm reminder tugas berpresisi tinggi).
   - Tidak meminta izin lokasi, kontak, SMS, atau kamera kecuali jika fitur spesifik di masa depan benar-benar membutuhkannya (misal: scan QR invite tim).
3. **Mekanisme Penghapusan Akun Lengkap (In-App & Web URL):**
   - Di dalam aplikasi: Settings > Akun > "Hapus Akun Saya" dengan konfirmasi kata sandi dan grace period 30 hari.
   - Di web publik: Endpoint URL publik (`https://kerjaiaja.com/delete-account` atau form web Supabase auth) yang memungkinkan pengguna meminta penghapusan data tanpa harus menginstal ulang aplikasi.
4. **Keamanan Kredensial & Secrets Management:**
   - Dilarang keras melakukan *hardcoding* API key atau token rahasia ke dalam source code Kotlin maupun repository Git.
   - Android client hanya memegang URL API publik dan Supabase Anon Key (yang dilindungi PostgreSQL RLS). Seluruh Service Role Key dan AI Backend API Key berada secara eksklusif di backend Go dan dilindungi environment variable.
5. **Kebijakan AI Generatif & Responsible AI:**
   - Setiap saran atau output teks yang dihasilkan AI menyertakan label penanda eksplisit ("Dihasilkan oleh AI").
   - Disediakan tombol feedback ("Laporkan saran tidak pantas / tidak relevan") untuk mematuhi regulasi Google Play Generative AI Policy.

## Konsekuensi
### Positif
- Memastikan kelancaran proses review Google Play Store tanpa penundaan atau risiko penolakan.
- Menjamin reputasi keamanan aplikasi di mata pengguna dan auditor.
- Meminimalkan permukaan serangan (*attack surface*) pada aplikasi mobile.

### Negatif / Mitigasi
- Membutuhkan pemeliharaan halaman kebijakan privasi dan alur penghapusan akun berbasis web publik sejak awal peluncuran.
  - *Mitigasi:* Dokumentasi panduan kesiapan Play Store disusun lengkap di `docs/security/PLAY_STORE_READINESS.md`.
