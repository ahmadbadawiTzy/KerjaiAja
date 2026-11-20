# ADR-007: Balanced MVP Scope

- **Status:** ACCEPTED
- **Tanggal:** 2026-11-20
- **Pengambil Keputusan:** Tim Arsitektur KerjaiAja (Senior Product Architect)
- **Terkait Dokumen:** [`docs/product/MVP_SCOPE.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/product/MVP_SCOPE.md), [`planning/IMPLEMENTATION_PHASES.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/planning/IMPLEMENTATION_PHASES.md)

---

## 1. Konteks (Context)
KerjaiAja memerlukan batasan ruang lingkup rilis perdana (MVP v1.0) yang seimbang (*balanced*): tidak terlalu sempit sehingga menjadi aplikasi to-do list generik tanpa daya saing, dan tidak terlalu luas sehingga gagal rilis (*development paralysis*).

## 2. Keputusan (Decision)
Menetapkan **Tiga Pilar Utama MVP yang Seimbang (Balanced MVP Scope)**:
1. **Pilar 1: Personal Academic Productivity:**
   - CRUD tugas akademik & aktivitas pribadi, kategori mata kuliah, deadline & kalender akademik sederhana, status, durasi estimasi, checklist verifikasi pengumpulan (*submission checklist*).
2. **Pilar 2: Intelligent Task Prioritization & AI Academic Assistant:**
   - Sistem penentuan urgensi hibrida (deterministik + kontekstual AI), kategori Critical/High/Medium/Low yang dapat dijelaskan (*explainable*), user override, dekonstruksi instruksi tugas (*breakdown*), rekomendasi tindakan harian, Task-Based RAG.
3. **Pilar 3: Team Collaboration:**
   - Pembuatan tim, undangan anggota via kode/link, penugasan tugas kelompok (*team tasks & assignee*), update status realtime, komentar/diskusi tugas tim, activity log tim.
4. **Fitur yang Secara Sadar Ditunda ke Pasca-MVP (v1.x / v2.0):**
   - Modul pencatatan materi mandiri multi-halaman (`academic_notes` CRUD).
   - OCR pemindaian silabus fisik / PDF attachment parser otomatis.
   - Enterprise project features (Gantt chart, time billing, dependencies visual graph kompleks).
   - Multi-platform (Web / iOS).

## 3. Konsekuensi (Consequences)
- **Positif:** Produk memiliki nilai diferensiasi tinggi sejak hari pertama; menyelesaikan siklus akademik pelajar secara utuh (tugas mandiri + kerja kelompok + panduan prioritas).
- **Negatif:** Membutuhkan koordinasi pengujian integrasi klien Android dengan backend Go dan database Supabase sejak awal.
