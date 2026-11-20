# Strategi Pengujian Web Admin CMS (CMS Test Strategy) — KerjaiAja
## Vitest Unit Testing · Supabase RLS Verification · Playwright E2E · Security Gate

- **Dokumen:** `docs/testing/CMS_TEST_STRATEGY.md`
- **Status:** BASELINE APPROVED
- **Test Runners:** Vitest (Unit & Integration), Playwright (E2E)
- **Cakupan:** Web Admin CMS & Supabase Security Policies

---

## 1. Piramida Pengujian Web Admin CMS

```text
          / \
         / E2E \         ~10% (Playwright: Auth Redirect, Login Flow, Navigation)
        /-------\
       / Integrasi \     ~30% (Supabase RLS Tests, Server Actions, RBAC Helpers)
      /   & RLS     \
     /---------------\
    /   Unit Testing  \  ~60% (Vitest: Zod Validation, Formatters, State Logic)
   /-------------------\
```

---

## 2. Rincian Lapisan Pengujian

### 2.1 Unit Tests (Vitest ~60%)
- **Target Pengujian:**
  1. **Validasi Skema Zod:**
     - Skema input penambahan kurikulum mata pelajaran (panjang karakter, format kode subjek unik).
     - Skema pembaruan status feedback pengguna (`NEW`, `IN_REVIEW`, `RESOLVED`, `DISMISSED`).
     - Validasi form login dan parameter paginasi.
  2. **Helper Evaluasi RBAC:**
     - Memverifikasi fungsi `hasPermission(role, action)` mengembalikan nilai boolean yang tepat.
     - Memastikan `SUPER_ADMIN` mewarisi seluruh hak wewenang.
     - Memastikan `SUPPORT` ditolak saat mencoba mutasi status akun atau kurikulum.
  3. **Data Formatters & Utility Functions:**
     - Format tanggal UTC ke waktu lokal Indonesia (WIB/WITA/WIT).
     - Format ukuran byte ke tampilan KB/MB/GB.
- **Pustaka:** `vitest`, `@testing-library/react`.

### 2.2 Integration & Supabase RLS Tests (~30%)
- **Target Pengujian:**
  1. **Uji Isolasi RLS (Row Level Security):**
     - Memverifikasi bahwa akun pengguna biasa (non-admin) ditolak saat mencoba melakukan `SELECT` pada tabel `admin_roles` atau `admin_audit_logs`.
     - Memverifikasi bahwa staf dengan peran `SUPPORT` tidak dapat melakukan `UPDATE` atau `DELETE` pada tabel `academic_references`.
     - Memverifikasi bahwa aksi admin berhasil tercatat di tabel `admin_audit_logs`.
  2. **Uji Server Actions:**
     - Memverifikasi penolakan eksekusi aksi server jika cookie sesi admin tidak valid.
     - Memverifikasi bahwa Server Action mengembalikan error terstruktur tanpa membocorkan pesan exception sistem.

### 2.3 End-to-End (E2E) Smoke Tests (Playwright ~10%)
- **Target Pengujian:**
  1. **Proteksi Akses Rute Tak Terotentikasi:**
     - Membuka URL `/admin/dashboard` tanpa login $\rightarrow$ wajib diarahkan (*redirect*) ke `/login`.
  2. **Alur Login Administrator:**
     - Mengisi form login dengan kredensial salah $\rightarrow$ menampilkan pesan error visual.
     - Mengisi kredensial admin valid $\rightarrow$ berhasil diarahkan ke `/admin/dashboard`.
  3. **Navigasi & Paginasi Dashboard:**
     - Berpindah antar menu sidebar (Dashboard, Users, Subjects, Feedback).
     - Menekan tombol paginasi tabel data (Next/Prev) dan memeriksa pembaruan parameter URL `?page=2`.
- **Pustaka:** `@playwright/test`.

---

## 3. Gerbang Verifikasi CI/CD (Quality & Security Gate)

Setiap pull request atau deployment produksi wajib melewati pemeriksaan otomatis:
```bash
# 1. Pemeriksaan Tipe Data Ketat TypeScript
pnpm tsc --noEmit

# 2. Linter dan Format Style
pnpm eslint . --max-warnings 0

# 3. Eksekusi Unit & Integration Tests
pnpm vitest run

# 4. Uji Keamanan Secret Leakage (Memastikan no NEXT_PUBLIC_SERVICE_ROLE_KEY)
pnpm run test:security
```
