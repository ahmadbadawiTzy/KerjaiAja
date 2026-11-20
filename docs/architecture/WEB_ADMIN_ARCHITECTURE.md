# Arsitektur Web Admin CMS (Web Admin Architecture) — KerjaiAja
## Next.js App Router · Supabase SSR Auth · Server Components · Tailwind CSS & shadcn/ui

- **Dokumen:** `docs/architecture/WEB_ADMIN_ARCHITECTURE.md`
- **Status:** BASELINE APPROVED
- **Tech Stack:** Next.js 15+, React 19, TypeScript (Strict), Tailwind CSS, shadcn/ui (Radix UI), `@supabase/ssr`
- **Hosting Target:** Vercel (Hobby / Self-hosted Container Compatible)
- **Shared Backend:** Supabase PostgreSQL Managed BaaS

---

## 1. Prinsip Desain Arsitektur Web

Arsitektur **KerjaiAja Web Admin CMS** dirancang dengan prinsip efisiensi komputasi, keamanan berlapis, dan pemisahan tanggung jawab yang bersih:

1. **Server-First by Default (React Server Components):**  
   Mayoritas halaman administratif mengambil data langsung di server (*Server Components*). Data dipanggil langsung dari Supabase melalui koneksi terotentikasi cookie, menghilangkan kebutuhan API boilerplate dan mengurangi ukuran bundle JavaScript yang dikirim ke browser.
2. **Strict TypeScript & Schema Validation (Zod):**  
   Setiap data input dari form atau parameter URL divalidasi secara ketat menggunakan Zod sebelum diproses di server.
3. **Pemisahan Klien & Server yang Bersih (`@supabase/ssr`):**  
   Koneksi Supabase menggunakan pustaka resmi `@supabase/ssr` dengan penanganan cookie sesi yang aman (*HttpOnly, Secure, SameSite*).
4. **Zero Client-Side Service Role Key:**  
   Kunci `SUPABASE_SERVICE_ROLE_KEY` hanya diakses pada operasi server terpilih yang sangat terisolasi (*Server Actions / Route Handlers*) dengan pemeriksaan wewenang `SUPER_ADMIN`. Browser hanya menerima token sesi terenkripsi.
5. **Responsif & Aksesibel:**  
   Desain antarmuka berbasis shadcn/ui dan Radix UI memastikan komponen memenuhi standar aksesibilitas keyboard WAI-ARIA dan nyaman digunakan di desktop maupun tablet.

---

## 2. Diagram Topologi & Aliran Data Ekosistem

```mermaid
flowchart TD
    subgraph ClientEnvironment ["Perangkat Klien"]
        AndroidApp["Aplikasi Android KerjaiAja\n(Kotlin + Room + WorkManager)"]
        AdminBrowser["Browser Administrator\n(Desktop / Tablet)"]
    end

    subgraph WebHostingZone ["Vercel Edge & Serverless Environment"]
        subgraph NextJsAppRouter ["Next.js 15 Web Admin CMS"]
            Middleware["Edge Middleware\n(Session Refresh & Route Guard)"]
            
            subgraph RouteSegments ["Segmentasi Rute"]
                PublicZone["/auth/login\n(Public Unauthenticated)"]
                ProtectedAdmin["/admin/*\n(Dashboard, Users, Subjects, Feedback)"]
            end
            
            subgraph DataLayer ["Lapisan Server Component & Action"]
                ServerComponents["React Server Components\n(SSR Direct Fetch via @supabase/ssr)"]
                ServerActions["Server Actions & Mutasi\n(Zod Validation & Audit Log Trigger)"]
            end
        end
    end

    subgraph SupabaseCloud ["Supabase Managed BaaS (Free Tier Rp0)"]
        SupaAuth["Supabase Auth Engine\n(Email/Password, Secure Cookies, JWT)"]
        
        subgraph SharedPostgres ["PostgreSQL Database (Single Source of Truth)"]
            AppTables[("Tabel Aplikasi Bersama\n(tasks, subjects, teams, feedback)")]
            AdminTables[("Tabel Administratif Khusus\n(admin_roles, admin_audit_logs)")]
            RLSPolicies["PostgreSQL Row Level Security (RLS)\n(auth.uid() & admin_roles check)"]
        end
        
        SupaStorage["Supabase Storage\n(Dokumen & Lampiran Tugas)"]
    end

    AndroidApp <-->|HTTPS REST & Realtime WS\n(anon_key + User JWT)| SupaAuth
    AndroidApp <-->|SyncWorker PostgREST (RLS)| AppTables
    
    AdminBrowser <-->|HTTPS Request / Responses| Middleware
    Middleware -->|Unauthenticated| PublicZone
    Middleware -->|Valid Admin Session| ProtectedAdmin
    
    ProtectedAdmin --> ServerComponents
    ProtectedAdmin --> ServerActions
    
    ServerComponents <-->|Secure SSR Cookie Client| SharedPostgres
    ServerActions <-->|Validated Mutasi + Audit| SharedPostgres
    ServerActions -. "Write Log" .-> AdminTables
    NextJsAppRouter <-->|Auth PKCE & Session Tokens| SupaAuth
```

---

## 3. Struktur Direktori Proyek CMS (`web-admin/`)

Struktur modular berbasis Next.js App Router:

```text
web-admin/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   │   └── page.tsx              # Halaman login administrator
│   │   └── layout.tsx                # Layout bersih tanpa sidebar
│   │
│   ├── (admin)/
│   │   ├── layout.tsx                # Shell layout: Sidebar, Header, Breadcrumbs
│   │   ├── dashboard/
│   │   │   └── page.tsx              # Ringkasan statistik operasional & kuota
│   │   ├── users/
│   │   │   ├── page.tsx              # Daftar pengguna, pencarian, status akun
│   │   │   └── [id]/page.tsx         # Detail audit akun pengguna (tanpa konten privat)
│   │   ├── subjects/
│   │   │   ├── page.tsx              # Manajemen 12 rumpun mata pelajaran & referensi
│   │   │   └── new/page.tsx          # Form penambahan silabus kurikulum
│   │   ├── teams/
│   │   │   └── page.tsx              # Pemantauan status tim & laporan pelanggaran
│   │   ├── feedback/
│   │   │   └── page.tsx              # Tiket keluhan, bug report, dan evaluasi OCR
│   │   └── audit-logs/
│   │       └── page.tsx              # Log aktivitas mutasi admin (Super Admin)
│   │
│   ├── api/
│   │   └── auth/
│   │       └── callback/route.ts     # Route handler penukaran kode PKCE auth
│   │
│   ├── globals.css                   # Tailwind tokens & tema
│   └── layout.tsx                    # Root HTML layout (Inter font, Providers)
│
├── components/
│   ├── ui/                           # Komponen primitif shadcn/ui (Button, Dialog, Table, Badge)
│   ├── admin-sidebar.tsx             # Navigasi utama dengan selektor peran
│   ├── admin-header.tsx              # Status user, tombol logout, breadcrumbs
│   ├── stat-card.tsx                 # Kartu metrik fungsional
│   └── data-table-pagination.tsx     # Komponen paginasi server-side
│
├── lib/
│   ├── supabase/
│   │   ├── client.ts                 # Browser client (createBrowserClient)
│   │   ├── server.ts                 # Server client (createServerClient dengan cookies)
│   │   ├── middleware.ts             # Session refresh helper untuk proxy edge
│   │   └── admin.ts                  # Privileged server client (Service Role terisolasi)
│   ├── rbac.ts                       # Helper validasi hak akses peran admin
│   └── validations/                  # Skema validasi Zod (users, subjects, feedback)
│
├── middleware.ts                     # Edge middleware proteksi rute /admin/*
├── tailwind.config.ts                # Konfigurasi token desain
├── tsconfig.json                     # Konfigurasi TypeScript strict mode
└── package.json                      # Dependensi Next.js, React 19, Supabase SSR
```

---

## 4. Pola Integrasi Supabase SSR (`@supabase/ssr`)

### 4.1 Server Client (`lib/supabase/server.ts`)
Mengakses Supabase dari Server Components, Route Handlers, dan Server Actions dengan menyertakan cookie sesi pengguna:

```typescript
import { createServerClient, type CookieOptions } from '@supabase/ssr'
import { cookies } from 'next/headers'

export async function createClient() {
  const cookieStore = await cookies()

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) {
          return cookieStore.get(name)?.value
        },
        set(name: string, value: string, options: CookieOptions) {
          try {
            cookieStore.set({ name, value, ...options })
          } catch (error) {
            // Handled when called from Server Component
          }
        },
        remove(name: string, value: string, options: CookieOptions) {
          try {
            cookieStore.set({ name, value: '', ...options })
          } catch (error) {
            // Handled when called from Server Component
          }
        },
      },
    }
  )
}
```

### 4.2 Edge Middleware Route Guarding (`middleware.ts`)
Memastikan sesi ter-refresh secara otomatis dan memblokir akses rute `/admin/*` jika pengguna belum login atau tidak memiliki peran admin:

```typescript
import { createServerClient, type CookieOptions } from '@supabase/ssr'
import { NextResponse, type NextRequest } from 'next/server'

export async function middleware(request: NextRequest) {
  let response = NextResponse.next({
    request: { headers: request.headers },
  })

  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) {
          return request.cookies.get(name)?.value
        },
        set(name: string, value: string, options: CookieOptions) {
          request.cookies.set({ name, value, ...options })
          response = NextResponse.next({
            request: { headers: request.headers },
          })
          response.cookies.set({ name, value, ...options })
        },
        remove(name: string, value: string, options: CookieOptions) {
          request.cookies.set({ name, value: '', ...options })
          response = NextResponse.next({
            request: { headers: request.headers },
          })
          response.cookies.set({ name, value: '', ...options })
        },
      },
    }
  )

  const { data: { user } } = await supabase.auth.getUser()

  // Lindungi seluruh rute yang berawalan /admin
  if (request.nextUrl.pathname.startsWith('/admin')) {
    if (!user) {
      return NextResponse.redirect(new URL('/login', request.url))
    }

    // Verifikasi peran admin dari tabel admin_roles
    const { data: adminRecord } = await supabase
      .from('admin_roles')
      .select('role, is_active')
      .eq('user_id', user.id)
      .eq('is_active', true)
      .single()

    if (!adminRecord) {
      // User login tetapi bukan administrator terdaftar
      return NextResponse.redirect(new URL('/login?error=unauthorized', request.url))
    }
  }

  return response
}

export const config = {
  matcher: ['/admin/:path*', '/login'],
}
```

---

## 5. Strategi Kinerja & Pengendalian Kuota Supabase

1. **Server-Side Pagination:**  
   Setiap query tabel di CMS menggunakan klausa `.range(from, to)` (misal 20–25 data per halaman) untuk mencegah penarikan data massal yang menguras memori dan kuota egress Supabase.
2. **Selective Column Projection:**  
   Dilarang menggunakan `select('*')` pada tabel besar. Selalu tentukan kolom yang diperlukan secara eksplisit (contoh: `select('id, name, created_at, status')`).
3. **No Unnecessary Realtime Subscriptions:**  
   CMS **TIDAK** membuka WebSocket Realtime secara global di semua halaman. Data diambil saat halaman dimuat (*pull on demand*), dengan tombol penyegaran manual (*refresh button*) jika administrator membutuhkan data terbaru.
4. **Cache Revalidation yang Tepat:**  
   Menggunakan `revalidatePath('/admin/subjects')` pada Server Actions setelah mutasi berhasil untuk memperbarui cache server secara instan.
