# ADR-037: CMS Role-Based Access Control and Server-Side Security Model
## Model Otorisasi Tiga Tingkat, Penegakan RLS, dan Audit Trail Administratif

- **Status:** APPROVED / LOCKED
- **Tanggal:** 2026-11-20
- **Penanggung Jawab:** Security Architect, Database Architect, Senior Full-Stack Web Engineer
- **Konteks:** Menjamin bahwa akses operasional ke Web Admin CMS dibatasi secara ketat, tidak ada kebocoran hak istimewa, dan setiap tindakan admin tercatat secara transparan.

---

## 1. Konteks & Masalah

Dashboard administratif mengelola data penting seperti kurikulum akademik, status tim, dan laporan bug pengguna. Kerentanan umum seperti Broken Access Control (OWASP Top 10 A01), kebocoran token service role, atau manipulasi hak akses via client-side metadata harus dicegah sejak fondasi arsitektur.

Kebutuhan sistem:
1. Membedakan wewenang antara pimpinan sistem, pengelola konten kurikulum, dan staf dukungan teknis.
2. Mencegah registrasi mandiri publik ke akun administrator.
3. Melarang pembacaan konten tugas personal atau foto soal mahasiswa oleh seluruh administrator.
4. Memastikan otorisasi ditegakkan di level basis data melalui RLS, bukan hanya visibilitas antarmuka pengguna.
5. Menyediakan jejak audit mutasi data (*audit trail*) yang tidak dapat dihapus.

---

## 2. Keputusan Arsitektur

Kami menetapkan arsitektur keamanan dan RBAC berikut:

1. **Model Peran Tiga Tingkat (Three-Tier RBAC):**
   - **`SUPER_ADMIN`:** Hak penuh atas sistem, manajemen staf admin, dan membaca log audit.
   - **`ADMIN`:** Mengelola referensi kurikulum, memoderasi nama tim kelompok, dan meninjau status pengguna.
   - **`SUPPORT`:** Merespon tiket feedback pengguna dan membaca laporan bug (akses read-only pada data kurikulum/tim).
2. **Isolasi Penyimpanan Peran (`public.admin_roles`):**
   - Peran disimpan di tabel relasional terpisah, **DILARANG** mengandalkan `raw_user_meta_data` auth Supabase karena dapat dimanipulasi oleh klien.
3. **Penegakan Hak Akses Berlapis (Defense in Depth):**
   - **Lapisan 1 (Edge Middleware):** Memblokir request tanpa sesi admin terverifikasi sebelum masuk ke halaman `/admin/*`.
   - **Lapisan 2 (PostgreSQL RLS):** Kueri database dilindungi fungsi `public.is_admin()` dan `public.has_admin_role()`.
   - **Lapisan 3 (Server Actions Validation):** Setiap aksi mutasi memvalidasi input dengan Zod dan memeriksa sesi server.
4. **Alur Pendaftaran Admin Tanpa Registrasi Publik (Invitation-Only):**
   - Form pendaftaran publik ditiadakan. Admin baru hanya dapat dibuat oleh `SUPER_ADMIN` melalui alur undangan terverifikasi atau bootstrapping SQL awal.
5. **Jejak Audit Mutasi Otomatis (`public.admin_audit_logs`):**
   - Seluruh operasi sensitif (suspend akun, publikasi materi, moderasi tim) wajib mencatat entri log *append-only* memuat admin ID, aksi, target, payload diff, dan timestamp UTC.
6. **Kedaulatan Konten Privat:**
   - Database RLS secara eksplisit melarang pembacaan catatan tugas pribadi pengguna oleh role administratif.

---

## 3. Konsekuensi

### Positif:
- **Tahan Terhadap Serangan Eskalasi Peran:** Klien tidak dapat mengubah peran mereka sendiri karena peran diverifikasi via tabel terlindungi di server.
- **Transparansi & Akuntabilitas:** Setiap perubahan operasional memiliki jejak audit yang jelas.
- **Kepatuhan Privasi Data (UU PDP):** Privasi akademik mahasiswa terlindungi dari pengintaian internal.

### Negatif / Mitigasi:
- **Overhead Query Verifikasi Peran:** Setiap request admin memerlukan evaluasi tabel `admin_roles`. Mitigasi: Indeks gabungan `(user_id, is_active, role)` dibuat untuk memastikan lookup berlangsung $< 1\text{ ms}$.

---

## 4. Referensi Dokumen Terkait

- [CMS Security and RBAC Architecture](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/CMS_SECURITY_AND_RBAC.md)
- [CMS Data Architecture](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/CMS_DATA_ARCHITECTURE.md)
- [Supabase Architecture](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/SUPABASE_ARCHITECTURE.md)
- [Security Review](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/security/SECURITY_REVIEW.md)
