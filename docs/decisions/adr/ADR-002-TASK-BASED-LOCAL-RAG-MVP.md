# ADR-002: Ruang Lingkup Local RAG MVP Berbasis Tugas (Task-Based Local RAG MVP Scope)

> [!NOTE]
> **Status:** **CONSOLIDATED INTO ADR-015 & ADR-039** (Rancangan Awal Pra-Rebaseline)  
> Dokumen ini merupakan rancangan awal ruang lingkup local RAG. Seluruh arsitektur pengambilan informasi dan pencarian leksikal telah dilebur dan diformalkan ke dalam:
> - **[ADR-015: Local RAG with SQLite FTS5 BM25](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-015-LOCAL-RAG-SQLITE-FTS5.md)**
> - **[ADR-039: Two-Tier Knowledge Intelligence Layer Architecture](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-039-TWO-TIER-KNOWLEDGE-INTELLIGENCE-LAYER.md)**  
> Nomor kanonikal ADR-002 pada master index dialokasikan untuk **[ADR-002: Go & Gin Modular Monolith Backend](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-002-GO-GIN-MODULAR-MONOLITH.md)** (Status: SUPERSEDED oleh ADR-016 & ADR-020).

- **Status:** CONSOLIDATED INTO ADR-015 & ADR-039
- **Tanggal:** 2026-11-20
- **Pengambil Keputusan:** Tim Arsitektur KerjaiAja (Disetujui Pengguna)
- **Terkait Dokumen:** [`docs/decisions/adr/ADR-015-LOCAL-RAG-SQLITE-FTS5.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-015-LOCAL-RAG-SQLITE-FTS5.md), [`docs/decisions/adr/ADR-039-TWO-TIER-KNOWLEDGE-INTELLIGENCE-LAYER.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-039-TWO-TIER-KNOWLEDGE-INTELLIGENCE-LAYER.md), [`docs/product/MVP_SCOPE.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/product/MVP_SCOPE.md)

---

## 1. Konteks (Context)

Untuk membantu model AI memberikan rekomendasi sub-tugas yang akurat dan kontekstual, arsitektur awal merancang subsistem *Local Retrieval-Augmented Generation (RAG)*. Pada dokumen awal, subsistem ini direncanakan mengambil konteks dari tabel catatan materi kuliah (`academic_notes`) menggunakan tabel virtual SQLite FTS (`academic_notes_fts`).

Audit dokumentasi (`docs/audits/SUPERPOWERS_DOCUMENTATION_AUDIT.md` temuan AUD-CRIT-02) menemukan bahwa modul CRUD untuk `academic_notes` sama sekali tidak pernah direncanakan di dalam `BACKLOG.md` maupun fase implementasi awal. Hal ini menyebabkan rantai pasok data RAG terputus (*orphaned data pipeline*).

## 2. Masalah (Problem)

Menambahkan modul pencatatan materi/silabus (`academic_notes`) secara mendadak ke dalam lingkup MVP v1.0 akan memperluas *surface area* rekayasa (*feature creep*), menambah layar baru, entitas baru, dan logika pengujian baru yang mengalihkan fokus dari modul utama: manajemen tugas akademik. Di sisi lain, membiarkan RAG tanpa sumber data akan menyebabkan fitur AI tidak berdaya guna.

## 3. Keputusan (Decision)

Menetapkan bahwa **Local RAG pada MVP (v1.0) hanya menggunakan konteks dari entitas `tasks`**:

1. **Penundaan Modul Catatan:** Modul `academic_notes` dan tabel virtual `academic_notes_fts` secara resmi **DITUNDA ke fase pasca-MVP (v1.x / v2.0)**.
2. **Konteks RAG Berbasis Tugas:** Konteks yang diinjeksikan ke dalam prompt asisten AI disusun secara eksklusif dari data tugas yang sudah tersedia:
   - Judul tugas (*Task Title*).
   - Deskripsi lengkap & instruksi pengerjaan tugas (*Task Description / Assignment Instructions*).
   - Nama mata kuliah & kode terkait (*Subject Name & Code*).
   - Atribut metadata tugas yang relevan (tipe tugas, durasi estimasi, skala prioritas).
3. **Pencarian Teks Penuh (FTS) Terfokus:** Tabel virtual SQLite FTS untuk MVP dialihkan menjadi `tasks_fts` (mengindeks teks judul dan deskripsi dari tabel `tasks`), digunakan untuk mencari riwayat tugas sejenis yang pernah dikerjakan sebelumnya sebagai contoh kontekstual (*few-shot context injection*).
4. **Prinsip Minimalisasi Data & Batas Akses AI:** Mesin AI hanya diberikan akses *read-only* ke blok teks tugas yang secara eksplisit dipilih pengguna. Model AI tidak diberikan izin akses query bebas (*unconstrained query access*) ke seluruh basis data Room.

## 4. Alternatif yang Dipertimbangkan (Alternatives Considered)

- **Alternatif A (Membuat Modul `academic_notes` di Sprint MVP):** Ditolak karena memperlambat rilis inti aplikasi dan menambah kompleksitas input pengguna.
- **Alternatif B (Menghapus RAG Sepenuhnya):** Ditolak karena instruksi tugas akademik sering kali panjang dan memerlukan penataan konteks terstruktur agar model 1B mampu memprosesnya dengan baik.

## 5. Konsekuensi (Consequences)

### Positif:
- Menghilangkan inkonsistensi antara arsitektur data dan backlog implementasi.
- Ruang lingkup MVP menjadi jauh lebih ramping, fokus, dan realistis untuk diselesaikan dalam sprint awal.
- Tidak membebani pengguna dengan keharusan mengetik catatan materi kuliah sebelum bisa menggunakan fitur breakdown AI.

### Negatif:
- Konteks pengetahuan AI terbatas pada teks yang dimasukkan pada kolom instruksi tugas, belum dapat mengaitkan materi silabus multi-halaman (hingga modul catatan diimplementasikan di v1.x).

## 6. Risiko & Mitigasi (Risks & Mitigations)

- **Risiko:** Pengguna hanya memasukkan instruksi tugas yang sangat singkat (misal: "Makalah AI").
- **Mitigasi:** Antarmuka input tugas memberikan petunjuk placeholder yang mendorong pengguna menempelkan (*paste*) poin-poin instruksi dosen secara lengkap.

## 7. Persyaratan Validasi (Validation Requirements)

- Memverifikasi skema Room `tasks_fts` beroperasi cepat (< 5 ms) saat mencari riwayat tugas sejenis.
