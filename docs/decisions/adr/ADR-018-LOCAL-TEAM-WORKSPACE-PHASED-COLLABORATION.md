# ADR-018: Ruang Kerja Tim Lokal & Penahapan Kolaborasi Multi-Perangkat (Phased Collaboration)

## Status
**ACCEPTED** (2026-11-20) — Menyesuaikan ([SUPERSEDES IN PART](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/ARCHITECTURE_DECISIONS.md)) [ADR-006](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-006-TEAM-COLLABORATION-IN-MVP.md) dan [ADR-007](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-007-BALANCED-MVP-SCOPE.md).

---

## 1. Konteks (Context)
Pilar ke-3 dari visi produk KerjaiAja adalah **Team Collaboration** (pengelolaan tugas kelompok mahasiswa). Pada arsitektur sebelumnya, fitur ini direncanakan mengandalkan backend Go dan Supabase Realtime WebSocket untuk sinkronisasi multi-perangkat instan.

Dengan dihapuskannya backend server dan dide-scope-nya Supabase demi tercapainya target **offline-first & biaya Rp0**, sistem tidak dapat lagi menjanjikan sinkronisasi real-time multi-perangkat pada rilis MVP v1.0. Menghapus pilar kolaborasi secara diam-diam akan mencederai visi produk, namun mengklaim fitur real-time tetap berjalan tanpa server adalah kebohongan teknis.

---

## 2. Keputusan (Decision)
KerjaiAja mengadopsi strategi **Penahapan Kolaborasi Transparan (Phased Collaboration Strategy)**:

1. **Pilar Kolaborasi Tetap Dipertahankan (Tidak Dihapus):**  
   Pengelolaan tugas kelompok tetap menjadi fitur inti aplikasi.
2. **Implementasi MVP v1.0 — Local Team Workspaces & File Sharing:**  
   - **Local Group Workspace:** Mahasiswa dapat membuat ruang kerja kelompok pada satu perangkat (misal: ponsel ketua kelompok), mendaftarkan nama anggota tim, memecah tugas besar menjadi bagian-bagian kerja, dan menetapkan penanggung jawab (*assignee*).
   - **File-Based Project Sharing:** Ketua kelompok dapat mengekspor berkas proyek kelompok (format file `.kerjai` atau JSON terkompresi) dan membagikannya via Android Share Sheet (WhatsApp, Telegram, Google Drive, Email).
   - **Import & Local Sync:** Anggota kelompok dapat mengimpor berkas tersebut ke aplikasi KerjaiAja di ponsel masing-masing untuk melihat tugas dan tenggat waktu mereka.
3. **Penjadwalan Cloud / P2P Sync ke Roadmap Pasca-MVP (v1.x / v2.0):**  
   Sinkronisasi otomatis lintas perangkat berbasis protokol P2P (Nearby Connections / WebRTC) atau relai sinkronisasi ringan tanpa server berbayar dijadwalkan secara resmi pada roadmap pasca-MVP setelah fondasi offline-first stabil.

---

## 3. Konsekuensi (Consequences)
### Positif:
- Menjaga transparansi teknis 100%: Tidak ada janji palsu terkait sinkronisasi real-time tanpa server.
- Fitur tugas kelompok tetap bermanfaat nyata bagi ketua kelompok yang ingin memetakan delegasi tugas anggotanya secara offline di ruang kelas.
- Bebas biaya server dan bebas risiko kegagalan koneksi jaringan saat presentasi tugas.
### Negatif:
- Perubahan status tugas oleh satu anggota kelompok tidak otomatis terupdate di ponsel anggota lain tanpa pengiriman ulang berkas proyek.
