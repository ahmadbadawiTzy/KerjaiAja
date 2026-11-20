# ADR-006: Team Collaboration as Core Pillar in MVP

- **Status:** ACCEPTED
- **Tanggal:** 2026-11-20
- **Pengambil Keputusan:** Tim Arsitektur KerjaiAja (Senior Product Architect)
- **Terkait Dokumen:** [`docs/product/MVP_SCOPE.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/product/MVP_SCOPE.md), [`docs/product/PRODUCT_REQUIREMENTS.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/product/PRODUCT_REQUIREMENTS.md), [`docs/architecture/BACKEND_ARCHITECTURE.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/BACKEND_ARCHITECTURE.md)

---

## 1. Konteks (Context)
Dalam kehidupan akademik mahasiswa dan pelajar, pengerjaan tugas kelompok (proyek praktikum, presentasi, makalah tim) menyumbang porsi besar dari beban studi dan merupakan sumber friksi utama (pembagian peran tidak merata, minim akuntabilitas, keterlambatan kolektif). Sebelumnya terdapat usulan audit untuk menunda modul kerja kelompok ke pasca-MVP v1.2. Pengguna secara eksplisit menolak penundaan ini dan menegaskan bahwa **Team Collaboration adalah pilar esensial yang wajib ada di MVP**.

## 2. Keputusan (Decision)
Mempertahankan dan mengimplementasikan **Team Collaboration sebagai salah satu dari Tiga Pilar Inti MVP (Balanced MVP)**:
1. **Fitur Tim MVP yang Terfokus (No Enterprise Bloat):**
   - **Pembuatan Tim & Undangan:** Mahasiswa dapat membuat tim studi dan mengundang teman sekelas via tautan atau kode unik (*invite code*).
   - **Manajemen Peran Sederhana:** Pemilik (*Owner/Creator*) dan Anggota (*Member*).
   - **Tugas Kelompok (*Team Tasks*):** Membuat tugas tim, menetapkan satu penanggung jawab utama (*assignee*), menentukan deadline, dan melacak status penyelesaian.
   - **Diskusi & Komentar Tugas:** Kolom komentar sederhana per tugas tim untuk koordinasi progres tanpa keluar aplikasi.
   - **Activity Log Tim:** Riwayat transparan mengenai aksi anggota (siapa yang menandai tugas selesai, siapa yang memperbarui deadline).
   - **Realtime Status Updates:** Menggunakan Supabase Realtime / WebSocket untuk sinkronisasi pembaruan status tugas tim secara instan antar-perangkat.
2. **Prinsip Keamanan & Privasi Antar-Anggota:**
   - Hak akses dievaluasi ketat di Go Backend: seorang anggota tim **TIDAK PERNAH** dapat melihat tugas pribadi (*personal tasks*) milik anggota lain.
   - Data tim diisolasi berdasarkan `team_id`.
   - Anggota hanya dapat memodifikasi tugas tim yang diizinkan oleh peran mereka.

## 3. Alternatif yang Dipertimbangkan (Alternatives Considered)
- **Menunda Kolaborasi ke v1.2:** Ditolak tegas oleh arahan produk pengguna; tugas kelompok adalah pain point mahasiswa nomor satu yang memberikan daya tarik organik (*viral loop*) paling kuat.
- **Enterprise Project Management Kompleks (Gantt Chart, Resource Allocation, Sprint Planning):** Ditolak untuk MVP; pendekatan Jira/Asana terlalu rumit untuk mahasiswa. Kolaborasi KerjaiAja difokuskan pada kejelasan peran (*role clarity*) dan akuntabilitas.

## 4. Konsekuensi (Consequences)
- **Positif:** Diferensiasi pasar yang sangat kuat; adopsi aplikasi menyebar secara alami dalam satu kelas kuliah; menyelesaikan masalah nyata *free-rider* dalam kerja kelompok.
- **Negatif:** Membutuhkan integrasi backend Go, autentikasi Supabase, dan kueri relasional multi-tenant sejak awal rilis MVP.
- **Implikasi Keamanan:** Wajib audit ketat otorisasi endpoint tim di backend Go (`RequireTeamMember` middleware).
- **Implikasi Biaya:** Tetap zero cost dengan skema relasional PostgreSQL Supabase Free Tier.
