# Manajemen Pengetahuan Kurikulum Pada Web Admin CMS
## Alur Kurasi, Persetujuan Publikasi, dan Distribusi Referensi Akademik

- **Status:** APPROVED & LOCKED
- **Lokasi Dokumen:** `docs/product/CMS_KNOWLEDGE_MANAGEMENT.md`
- **Terkait:** [`CMS_PRODUCT_REQUIREMENTS.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/product/CMS_PRODUCT_REQUIREMENTS.md), [`CMS_DATA_ARCHITECTURE.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/CMS_DATA_ARCHITECTURE.md)

---

## 1. Peran & Batasan Arsitektur CMS

Web Admin CMS bertindak sebagai **pusat kendali kurasi materi kurikulum akademik** yang digunakan oleh aplikasi Android KerjaiAja. CMS memastikan hanya materi yang telah melalui proses kurasi dan verifikasi yang didistribusikan ke perangkat siswa.

> [!IMPORTANT]
> **Batasan Privasi & Keamanan Tegas:**
> - CMS murni mengelola data kurikulum publik (`academic_references` dan `curriculum_topics`).
> - CMS **DILARANG KERAS** memiliki fitur untuk meninjau, mengintip, atau mencari catatan tugas, jadwal, atau foto soal privat milik mahasiswa (*Zero Student Surveillance Policy*).

---

## 2. Siklus Hidup Publikasi Konten (Content Publishing Workflow)

Setiap materi kurikulum baru wajib melewati tahapan persetujuan formal sebelum dapat disinkronkan ke aplikasi Android:

```
[Unggah Dokumen / Teks Baru]
            |
            v
       +----------+
       |  DRAFT   | -> Dokumen sedang diedit; TIDAK disinkronkan ke Android
       +----------+
            |
            v (Submit for Review)
       +----------+
       | IN_REVIEW| -> Ditinjau oleh kurator akademik / Admin
       +----------+
            |
            +------------+------------+
            |                         |
   (Ditolak/Revisi)            (Disetujui)
            v                         v
       +----------+             +-----------+
       | REJECTED |             | PUBLISHED | -> Tersedia untuk Delta Sync Android
       +----------+             +-----------+
                                      |
                              (Penarikan Materi)
                                      v
                                +-----------+
                                |  REVOKED  | -> Deletion marker disinkronkan ke Android
                                +-----------+
```

### Definisi Status Dokumen:
1. **`DRAFT`:** Materi baru dibuat oleh staf kurator. Hanya dapat dilihat oleh pembuat dan `ADMIN`. Tidak memiliki status publikasi.
2. **`IN_REVIEW`:** Materi diajukan untuk ditinjau kelayakan pedagogis, keakuratan rumus, dan kelengkapan sitasi sumbernya.
3. **`PUBLISHED`:** Materi disetujui oleh `ADMIN` atau `SUPER_ADMIN`. Flag `is_published = true` diaktifkan, dan kolom `updated_at = NOW()` diperbarui. Dokumen otomatis masuk ke antrean delta sync klien Android.
4. **`REVOKED`:** Materi ditarik dari peredaran (misal karena ada revisi kurikulum resmi). Flag `is_published = false` dikirim ke klien Android untuk dihapus dari basis pencarian lokal.

---

## 3. Fitur Utama Modul Pengetahuan di CMS (Scope MVP)

### 3.1 Manajemen Dokumen & Kurikulum (Must Have - P0)
- **CMS-KM-01: Unggah & Editor Materi:** Admin dapat membuat entri referensi baru (format Markdown atau teks terstruktur), menentukan rumpun mata pelajaran (12 mapel), kategori materi, dan sumber buku referensi.
- **CMS-KM-02: Pratinjau Chunking:** Antarmuka CMS menampilkan pratinjau bagaimana teks akan dipotong (*chunking preview*) untuk memastikan rumus dan judul bab tidak terpotong sembarangan.
- **CMS-KM-03: Riwayat Versi & Hash Konten:** Sistem menghitung hash SHA-256 berkas untuk mencegah unggahan materi duplikat dan menaikkan nomor versi (`document_version`) setiap kali ada perbaikan isi.

### 3.2 Workbench Pengujian Pencarian (Should Have - P1)
- **CMS-KM-04: Search Testing Sandbox:** Halaman khusus di mana admin dapat mengetikkan kueri pencarian (misal: *"rumus gaya gesek"*) dan melihat daftar chunk materi yang akan muncul pada ranking teratas beserta estimasi skor BM25. Hal ini memungkinkan admin memverifikasi efektivitas pencarian sebelum materi dipublikasikan ke siswa.

---

## 4. Skema Tabel Pengetahuan di Supabase PostgreSQL

```sql
-- Tabel Master Referensi Kurikulum di Supabase
CREATE TABLE IF NOT EXISTS public.academic_references (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title TEXT NOT NULL,
    subject TEXT NOT NULL,
    category TEXT NOT NULL,
    source_reference TEXT NOT NULL,
    content TEXT NOT NULL,
    version INTEGER NOT NULL DEFAULT 1,
    content_hash TEXT NOT NULL,
    status TEXT NOT NULL DEFAULT 'DRAFT' CHECK (status IN ('DRAFT', 'IN_REVIEW', 'PUBLISHED', 'REVOKED')),
    is_published BOOLEAN NOT NULL DEFAULT FALSE,
    created_by UUID REFERENCES auth.users(id),
    approved_by UUID REFERENCES auth.users(id),
    created_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc'::text, now()),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc'::text, now())
);

-- Indeks efisiensi Delta Sync
CREATE INDEX IF NOT EXISTS idx_academic_refs_sync 
ON public.academic_references (updated_at) 
WHERE is_published = TRUE;
```
