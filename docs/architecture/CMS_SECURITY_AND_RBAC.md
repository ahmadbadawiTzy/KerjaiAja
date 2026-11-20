# Arsitektur Keamanan & RBAC Web Admin CMS (Security & RBAC Architecture) — KerjaiAja
## Least Privilege · Supabase RLS · Tamper-Evident Audit Trail · Zero Trust Administration

- **Dokumen:** `docs/architecture/CMS_SECURITY_AND_RBAC.md`
- **Status:** BASELINE APPROVED
- **Standar Keamanan:** OWASP Top 10 (2025), OWASP MASVS, Supabase Security Best Practices
- **Prinsip Utama:** *Defense in Depth*, *Zero Trust*, *No Client Service Role*, *Strict Least Privilege*

---

## 1. Filosofi Keamanan & Batasan Akses

KerjaiAja memegang prinsip kedaulatan data dan privasi mutlak bagi pelajar dan mahasiswa:
1. **Hak Istimewa Terkecil (Least Privilege):** Setiap staf administratif hanya diberikan izin akses minimum yang mutlak diperlukan untuk menyelesaikan tugas operasionalnya.
2. **Kedaulatan Konten Privat (Surveillance Ban):** Hak akses administratif **TIDAK PERNAH** mencakup kemampuan mengintip isi catatan tugas pribadi, foto dokumen soal, atau percakapan pribadi antar-mahasiswa.
3. **Validasi Otorisasi di Sisi Server (Server-Side Enforcement):** Proteksi hak akses ditegakkan di level database (PostgreSQL Row Level Security) dan Next.js Middleware/Server Actions. Menyembunyikan tombol di antarmuka (UI visibility) **BUKAN** mekanisme keamanan.
4. **Jejak Audit Tidak Dapat Dimanipulasi (Tamper-Evident Audit Logging):** Setiap tindakan perubahan atau penghapusan data wajib dicatat secara otomatis dalam log audit yang bersifat *append-only*.
5. **Larangan Registrasi Mandiri Publik (No Public Self-Registration):** Administrator baru hanya dapat didaftarkan melalui proses undangan resmi (*provisioning*) oleh `SUPER_ADMIN`.

---

## 2. Model Peran (Role-Based Access Control Matrix)

Tiga peran administratif didefinisikan secara eksplisit:

| Izin Operasional / Sumber Daya | `SUPER_ADMIN` | `ADMIN` | `SUPPORT` |
| :--- | :---: | :---: | :---: |
| **Login ke Dashboard Web CMS** | ✅ Ya | ✅ Ya | ✅ Ya |
| **Melihat Metrik Dashboard Umum** | ✅ Ya | ✅ Ya | ✅ Ya |
| **Melihat Daftar Pengguna (Metadata)** | ✅ Ya | ✅ Ya | 🟡 Terbatas (Pencarian ID) |
| **Mengubah Status Akun Pengguna (Suspend/Ban)** | ✅ Ya | ✅ Ya | ❌ Ditolak |
| **Menghapus Akun Permanen (Right to Erasure)** | ✅ Ya | ❌ Ditolak | ❌ Ditolak |
| **Kelola 12 Rumpun Mata Pelajaran & Silabus** | ✅ Ya | ✅ Ya | 🟡 Read-Only |
| **Meninjau & Memoderasi Ruang Kerja Tim** | ✅ Ya | ✅ Ya | 🟡 Read-Only |
| **Merespon & Menyelesaikan Tiket Feedback** | ✅ Ya | ✅ Ya | ✅ Ya |
| **Mengelola Staf Admin (Invite/Role Assign)** | ✅ Ya | ❌ Ditolak | ❌ Ditolak |
| **Membaca Log Audit (`admin_audit_logs`)** | ✅ Ya | ❌ Ditolak | ❌ Ditolak |
| **Melihat Catatan/Foto Pribadi Pengguna** | ❌ **DILARANG MUTLAK BAGI SEMUA PERAN** | ❌ **DILARANG** | ❌ **DILARANG** |

---

## 3. Implementasi Keamanan Database PostgreSQL

### 3.1 Skema Tabel Peran Administratif (`public.admin_roles`)
Peran tidak disimpan di dalam `user_metadata` auth (karena dapat dimanipulasi oleh pengguna melalui API klien), melainkan disimpan di tabel terisolasi dengan RLS aktif:

```sql
-- ENUM Tipe Peran Admin
CREATE TYPE public.admin_role_type AS ENUM ('SUPER_ADMIN', 'ADMIN', 'SUPPORT');

-- Tabel Relasi Peran Admin
CREATE TABLE public.admin_roles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    role public.admin_role_type NOT NULL DEFAULT 'SUPPORT',
    is_active BOOLEAN NOT NULL DEFAULT true,
    created_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc', now()),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc', now()),
    created_by UUID REFERENCES auth.users(id),
    CONSTRAINT unique_admin_user UNIQUE (user_id)
);

-- Indeks untuk lookup cepat di RLS
CREATE INDEX idx_admin_roles_lookup ON public.admin_roles (user_id, is_active, role);

-- Aktifkan RLS
ALTER TABLE public.admin_roles ENABLE ROW LEVEL SECURITY;
```

### 3.2 Fungsi Evaluasi Keamanan (`SECURITY DEFINER` dengan Proteksi)
Sesuai panduan keamanan Supabase, fungsi helper yang membaca status admin wajib diisolasi dan menetapkan `search_path`:

```sql
-- Fungsi helper untuk memeriksa apakah user saat ini adalah admin aktif
CREATE OR REPLACE FUNCTION public.is_admin()
RETURNS BOOLEAN
LANGUAGE sql
STABLE
SECURITY DEFINER
SET search_path = public
AS $$
    SELECT EXISTS (
        SELECT 1 
        FROM public.admin_roles 
        WHERE user_id = auth.uid() 
          AND is_active = true
    );
$$;

-- Fungsi helper untuk memeriksa role spesifik
CREATE OR REPLACE FUNCTION public.has_admin_role(required_role public.admin_role_type)
RETURNS BOOLEAN
LANGUAGE sql
STABLE
SECURITY DEFINER
SET search_path = public
AS $$
    SELECT EXISTS (
        SELECT 1 
        FROM public.admin_roles 
        WHERE user_id = auth.uid() 
          AND is_active = true
          AND (
              role = required_role 
              OR role = 'SUPER_ADMIN' -- SUPER_ADMIN mewarisi semua hak
          )
    );
$$;

-- Cabut eksekusi dari publik dan izinkan hanya untuk role terotentikasi
REVOKE EXECUTE ON FUNCTION public.is_admin() FROM PUBLIC;
GRANT EXECUTE ON FUNCTION public.is_admin() TO authenticated;

REVOKE EXECUTE ON FUNCTION public.has_admin_role(public.admin_role_type) FROM PUBLIC;
GRANT EXECUTE ON FUNCTION public.has_admin_role(public.admin_role_type) TO authenticated;
```

### 3.3 Kebijakan RLS Tabel Administratif
```sql
-- Hanya SUPER_ADMIN yang dapat membaca seluruh daftar admin
CREATE POLICY "super_admin_select_admin_roles"
ON public.admin_roles
FOR SELECT
TO authenticated
USING (public.has_admin_role('SUPER_ADMIN'));

-- Hanya SUPER_ADMIN yang dapat mengelola staf admin
CREATE POLICY "super_admin_manage_admin_roles"
ON public.admin_roles
FOR ALL
TO authenticated
USING (public.has_admin_role('SUPER_ADMIN'))
WITH CHECK (public.has_admin_role('SUPER_ADMIN'));
```

---

## 4. Log Audit Mutasi Data (`public.admin_audit_logs`)

Setiap tindakan perubahan status akun, publikasi silabus, atau moderasi tim wajib meninggalkan jejak audit:

```sql
CREATE TABLE public.admin_audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    admin_id UUID NOT NULL REFERENCES auth.users(id),
    action VARCHAR(64) NOT NULL,            -- e.g. 'SUSPEND_USER', 'RESOLVE_FEEDBACK', 'PUBLISH_SUBJECT'
    target_table VARCHAR(64) NOT NULL,      -- e.g. 'users', 'user_feedbacks', 'academic_references'
    target_id VARCHAR(128) NOT NULL,        -- ID dari baris yang diubah
    payload JSONB,                          -- Snapshot perubahan (diff)
    ip_address VARCHAR(45),                 -- Alamat IP klien (sanitized/hashed)
    user_agent TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc', now())
);

-- Indeks audit log untuk investigasi cepat
CREATE INDEX idx_audit_logs_timestamp ON public.admin_audit_logs (created_at DESC);
CREATE INDEX idx_audit_logs_admin ON public.admin_audit_logs (admin_id);

-- Aktifkan RLS
ALTER TABLE public.admin_audit_logs ENABLE ROW LEVEL SECURITY;

-- Hanya SUPER_ADMIN yang dapat membaca log audit (Append-only untuk semua admin)
CREATE POLICY "super_admin_view_audit_logs"
ON public.admin_audit_logs
FOR SELECT
TO authenticated
USING (public.has_admin_role('SUPER_ADMIN'));

-- Seluruh admin terverifikasi dapat mencatat log audit (INSERT ONLY)
CREATE POLICY "admins_insert_audit_logs"
ON public.admin_audit_logs
FOR INSERT
TO authenticated
WITH CHECK (public.is_admin());
```

---

## 5. Safeguards Arsitektur & Pencegahan Kerentanan

1. **Anti-IDOR (Insecure Direct Object References):**
   - RLS memastikan klien web tidak dapat memanipulasi `user_id` atau ID entitas lain di luar kewenangannya.
2. **Pemberian Hak Terkendali (No Broad Grants):**
   - Hak `UPDATE` pada tabel `academic_references` memerlukan kebijakan `USING` dan `WITH CHECK` yang identik untuk mencegah pembajakan kepemilikan.
3. **Penyembunyian Kredensial Rahasia:**
   - Variabel lingkungan `SUPABASE_SERVICE_ROLE_KEY` **DILARANG KERAS** menggunakan prefix `NEXT_PUBLIC_`.
   - Pemeriksaan linter dan build CI memvalidasi ketiadaan kebocoran token pada berkas client bundle.
4. **Session Timeout & Invalidation:**
   - Masa berlaku token JWT admin dibatasi maksimal 1 jam dengan refresh token otomatis via cookie HttpOnly.
   - Penonaktifan akun admin di tabel `admin_roles` (`is_active = false`) langsung mencabut akses pada request berikutnya melalui evaluasi middleware dan RLS.
