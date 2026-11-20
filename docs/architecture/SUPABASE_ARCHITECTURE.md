# ARSITEKTUR SUPABASE MANAGED BAAS — KERJAIAJA
## Direct Client Integration via Supabase Kotlin Client (`supabase-kt`)

**Dokumen:** `docs/architecture/SUPABASE_ARCHITECTURE.md`  
**Status:** Baseline Disetujui  
**Target Platform:** Android Native (Kotlin)  
**Client Library:** `io.github.jan-tennert.supabase:bom:3.9.0`  

---

## 1. Ikhtisar & Prinsip Arsitektur

KerjaiAja memanfaatkan **Supabase** secara eksklusif sebagai Backend-as-a-Service (BaaS) terkelola untuk menangani kebutuhan cloud: autentikasi, penyimpanan data terpusat, sinkronisasi antar-perangkat, dan kolaborasi tugas kelompok.

### Prinsip Utama:
1. **Direct Android Client Connection:** Aplikasi Android berkomunikasi langsung dengan REST PostgREST dan WebSocket Realtime Supabase menggunakan kunci publik anonim (`anon_key`). Tidak ada perantara server custom (Go, Node.js, atau Python).
2. **PostgreSQL Row Level Security (RLS) as Security Gateway:** Seluruh aturan otorisasi dan isolasi data ditegakkan di level database PostgreSQL menggunakan klausa `auth.uid()`.
3. **Zero AI Compute in Cloud:** Supabase tidak digunakan untuk mengeksekusi inferensi AI, prompt routing, atau penyimpanan model weights AI.

---

## 2. Pustaka & Konfigurasi Supabase Client

Aplikasi Android menggunakan Supabase Kotlin SDK v3.9.0 dengan Ktor 3.x Android Engine:

```kotlin
// Inisialisasi SupabaseClient di Dependency Injection (Hilt / Singleton)
val supabase = createSupabaseClient(
    supabaseUrl = BuildConfig.SUPABASE_URL,
    supabaseKey = BuildConfig.SUPABASE_ANON_KEY
) {
    install(Auth) {
        flowType = FlowType.PKCE
        scheme = "kerjaiaja"
        host = "login-callback"
    }
    install(Postgrest)
    install(Realtime)
    install(Storage) {
        transferTimeout = 30.seconds
    }
    httpEngine = Android.create()
}
```

---

## 3. Skema Database PostgreSQL

Skema dirancang relasional, efisien secara ukuran (storage-optimized), dan dilengkapi indeks untuk performa tinggi dalam batas Free Tier:

```sql
-- 1. PROFILES (Extends auth.users)
CREATE TABLE public.profiles (
    id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
    email TEXT NOT NULL,
    full_name TEXT NOT NULL,
    avatar_url TEXT,
    academic_institution TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc', now()),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc', now())
);

-- 2. SUBJECTS (Mata Kuliah / Pelajaran)
CREATE TABLE public.subjects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
    name TEXT NOT NULL,
    code TEXT,
    color_hex TEXT NOT NULL DEFAULT '#1E293B',
    instructor_name TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc', now()),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc', now()),
    deleted_at TIMESTAMPTZ
);

-- 3. TASKS (Tugas Pribadi)
CREATE TABLE public.tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
    subject_id UUID REFERENCES public.subjects(id) ON DELETE SET NULL,
    title TEXT NOT NULL,
    description TEXT,
    deadline TIMESTAMPTZ NOT NULL,
    estimated_minutes INT NOT NULL DEFAULT 60,
    difficulty INT NOT NULL DEFAULT 3 CHECK (difficulty BETWEEN 1 AND 5),
    grade_weight DOUBLE PRECISION NOT NULL DEFAULT 10.0,
    is_completed BOOLEAN NOT NULL DEFAULT FALSE,
    completed_at TIMESTAMPTZ,
    deterministic_priority INT NOT NULL DEFAULT 50,
    created_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc', now()),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc', now()),
    deleted_at TIMESTAMPTZ,
    version INT NOT NULL DEFAULT 1
);

-- 4. SUBTASKS (Daftar Checklist Tugas)
CREATE TABLE public.subtasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    task_id UUID NOT NULL REFERENCES public.tasks(id) ON DELETE CASCADE,
    title TEXT NOT NULL,
    is_completed BOOLEAN NOT NULL DEFAULT FALSE,
    sort_order INT NOT NULL DEFAULT 0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc', now()),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc', now()),
    deleted_at TIMESTAMPTZ
);

-- 5. TEAMS (Kelompok Belajar / Proyek)
CREATE TABLE public.teams (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    description TEXT,
    created_by UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc', now()),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc', now()),
    deleted_at TIMESTAMPTZ
);

-- 6. TEAM MEMBERS (Keanggotaan & Role)
CREATE TABLE public.team_members (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    team_id UUID NOT NULL REFERENCES public.teams(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
    role TEXT NOT NULL DEFAULT 'member' CHECK (role IN ('owner', 'admin', 'member')),
    joined_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc', now()),
    UNIQUE(team_id, user_id)
);

-- 7. TEAM TASKS (Tugas Kolaboratif Tim)
CREATE TABLE public.team_tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    team_id UUID NOT NULL REFERENCES public.teams(id) ON DELETE CASCADE,
    assigned_to UUID REFERENCES public.profiles(id) ON DELETE SET NULL,
    title TEXT NOT NULL,
    description TEXT,
    deadline TIMESTAMPTZ NOT NULL,
    status TEXT NOT NULL DEFAULT 'todo' CHECK (status IN ('todo', 'in_progress', 'review', 'done')),
    created_by UUID NOT NULL REFERENCES public.profiles(id),
    created_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc', now()),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc', now()),
    deleted_at TIMESTAMPTZ,
    version INT NOT NULL DEFAULT 1
);
```

---

## 4. Kebijakan Row Level Security (RLS)

Setiap tabel diaktifkan RLS (`ALTER TABLE ... ENABLE ROW LEVEL SECURITY;`).

### Aturan Data Pribadi (`profiles`, `subjects`, `tasks`, `subtasks`):
Pengguna hanya dapat melihat dan memodifikasi data milik mereka sendiri:
```sql
CREATE POLICY "Users can manage their own profiles"
ON public.profiles FOR ALL USING (auth.uid() = id);

CREATE POLICY "Users can manage their own subjects"
ON public.subjects FOR ALL USING (auth.uid() = user_id);

CREATE POLICY "Users can manage their own tasks"
ON public.tasks FOR ALL USING (auth.uid() = user_id);

CREATE POLICY "Users can manage their own subtasks"
ON public.subtasks FOR ALL USING (
    EXISTS (SELECT 1 FROM public.tasks WHERE tasks.id = subtasks.task_id AND tasks.user_id = auth.uid())
);
```

### Aturan Kolaborasi Tim (`teams`, `team_members`, `team_tasks`):
Hanya anggota tim yang terverifikasi yang dapat membaca dan memperbarui data kelompok:
```sql
CREATE POLICY "Members can view teams they belong to"
ON public.teams FOR SELECT USING (
    EXISTS (SELECT 1 FROM public.team_members WHERE team_members.team_id = teams.id AND team_members.user_id = auth.uid())
);

CREATE POLICY "Members can view and edit team tasks"
ON public.team_tasks FOR ALL USING (
    EXISTS (SELECT 1 FROM public.team_members WHERE team_members.team_id = team_tasks.team_id AND team_members.user_id = auth.uid())
);
```

- **Bucket Name:** `task_attachments` (Private bucket).
- **Penggunaan:** Hanya untuk lampiran dokumen tugas akademik pengguna (PDF, DOCX, JPEG/PNG).
- **Batas Ukuran:** Maksimal **5 MB per berkas**.
- **Larangan Keras:** Model weights on-device Gemma (.bin/.litertlm) dilarang disimpan di Supabase Storage.
- **Kebijakan Akses (Storage RLS):**
  Pengguna hanya dapat mengunggah dan mengunduh berkas di path direktori yang diawali dengan ID pengguna mereka: `(storage.foldername(name))[1] = auth.uid()::text`.

---

## 6. Integrasi Ekosistem Web Admin CMS

Basis data Supabase yang sama melayani **KerjaiAja Web Admin CMS** via `@supabase/ssr` (Next.js 15).

### 6.1 Tabel Administratif & Kurikulum Bersama
1. **`public.admin_roles`:** Menyimpan pemetaan peran staf (`SUPER_ADMIN`, `ADMIN`, `SUPPORT`).
2. **`public.academic_references` & `public.curriculum_topics`:** Master referensi 12 mata pelajaran yang dikelola admin dan dibaca oleh Android via Delta Sync.
3. **`public.user_feedbacks`:** Menampung keluhan dan bug report dari aplikasi Android untuk direspon oleh admin.
4. **`public.admin_audit_logs`:** Mencatat jejak audit setiap aksi perubahan data administratif.

### 6.2 Aturan RLS Khusus Admin
```sql
-- Helper fungsi otorisasi di database
CREATE OR REPLACE FUNCTION public.is_admin()
RETURNS BOOLEAN LANGUAGE sql STABLE SECURITY DEFINER SET search_path = public AS $$
    SELECT EXISTS (SELECT 1 FROM public.admin_roles WHERE user_id = auth.uid() AND is_active = true);
$$;

-- Admin dapat mengelola referensi kurikulum akademik
CREATE POLICY "Admins manage academic references"
ON public.academic_references FOR ALL TO authenticated
USING (public.is_admin()) WITH CHECK (public.is_admin());

-- Pengguna Android & Web dapat membaca referensi kurikulum yang telah dipublikasikan
CREATE POLICY "Public read published academic references"
ON public.academic_references FOR SELECT TO authenticated, anon
USING (is_published = true);

-- Admin dapat membaca dan memperbarui status feedback
CREATE POLICY "Admins manage feedback"
ON public.user_feedbacks FOR ALL TO authenticated
USING (public.is_admin()) WITH CHECK (public.is_admin());
```

---

## 7. Referensi Putusan Arsitektur Terkait

- [ADR-020: Supabase Managed BaaS Integration](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-020-SUPABASE-MANAGED-BAAS.md)
- [ADR-036: Web Admin CMS Architecture and Technology Stack](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-036-WEB-ADMIN-CMS-ARCHITECTURE.md)
- [ADR-037: CMS Role-Based Access Control & Server-Side Security](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-037-CMS-RBAC-AND-SERVER-SIDE-SECURITY.md)
- [ADR-038: Shared Database Schema & Multi-Tenant RLS for CMS](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-038-SHARED-DATABASE-SCHEMA-AND-CMS-RLS.md)

