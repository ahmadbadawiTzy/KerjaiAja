# ADR-017: Arsitektur Data Offline-First Murni & Penundaan Supabase Cloud (Descoping)

## Status
**ACCEPTED** (2026-11-20) — Menggantikan ([SUPERSEDES](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/ARCHITECTURE_DECISIONS.md)) [ADR-003](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-003-SUPABASE-MANAGED-INFRASTRUCTURE.md).

---

## 1. Konteks (Context)
Sebelumnya, Supabase direncanakan sebagai penyedia infrastruktur terkelola (*managed backend*) untuk autentikasi pengguna, penyimpanan data relasional cloud (PostgreSQL), dan sinkronisasi status tim realtime via WebSocket.

Namun, ketergantungan pada database cloud memaksa aplikasi untuk selalu terhubung ke internet saat autentikasi dan pembuatan akun, bertentangan dengan prinsip **offline-first & local-first** di mana mahasiswa harus dapat langsung menggunakan aplikasi seketika setelah instalasi tanpa harus login atau menunggu verifikasi email.

---

## 2. Keputusan (Decision)
1. **Descoping Supabase dari Ruang Lingkup MVP v1.0:**  
   Menghapus ketergantungan client library Supabase, remote schema migrations, dan otentikasi Supabase cloud dari target rilis MVP v1.0.  
   *(Catatan: Proyek Supabase remote yang telah ada di cloud tidak dihapus secara destruktif, melainkan dinonaktifkan dari alur runtime MVP).*
2. **Local-First Persistence Stack:**  
   - **Room Database (SQLite):** Menjadi *Single Source of Truth* untuk seluruh entitas personal: tugas, sub-tugas, mata kuliah, jadwal kelas, dan proyek kelompok lokal.
   - **Jetpack DataStore:** Menyimpan preferensi UI, status unduhan model AI, dan konfigurasi lokal.
   - **Android Keystore:** Mengamankan material enkripsi lokal.
3. **Pemberian Hak Penggunaan Instan (Instant App Usability):**  
   Pengguna baru tidak diwajibkan mendaftarkan akun atau login cloud. Aplikasi langsung siap pakai sesaat setelah dibuka pertama kali.
4. **Pencadangan & Ekspor Mandiri (Self-Managed Backup & Export):**  
   Menyediakan fitur ekspor dan impor basis data lokal dalam format JSON / SQLite terenkripsi sehingga pengguna tetap memiliki kendali penuh atas migrasi perangkat.

---

## 3. Konsekuensi (Consequences)
### Positif:
- Aplikasi dapat dibuka dan digunakan 100% tanpa sinyal internet sejak detik pertama.
- Tidak ada latency sinkronisasi jaringan saat membuat atau memperbarui tugas.
- Pengguna memiliki kepemilikan data penuh (*data sovereignty*).
### Negatif:
- Tidak ada pemulihan akun otomatis lintas perangkat berbasis cloud jika ponsel pengguna hilang tanpa sempat melakukan ekspor cadangan lokal.
