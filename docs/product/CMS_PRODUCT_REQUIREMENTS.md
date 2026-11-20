# Spesifikasi Kebutuhan Produk Web Admin CMS (CMS Product Requirements) — KerjaiAja
## Pusat Tata Kelola Operasional Produk & Dukungan Akademik Berbasis Web

- **Dokumen:** `docs/product/CMS_PRODUCT_REQUIREMENTS.md`
- **Status:** BASELINE APPROVED (Hasil Superpowers Product Discovery)
- **Target Platform:** Web (Next.js 15 App Router, React 19, TypeScript Strict, Tailwind CSS, shadcn/ui)
- **Deployment:** Vercel (Hobby / Self-hosted Compatible)
- **Backend Terpadu:** Supabase Managed BaaS (PostgreSQL, Auth, RLS, Storage)

---

## 1. Latar Belakang & Penyelarasan Visi Produk

Platform **KerjaiAja** hadir untuk membantu pelajar dan mahasiswa mengelola tugas akademik secara terstruktur, kolaboratif, dan adaptif tanpa rasa bersalah (*anti-guilt UX*). Aplikasi Android beroperasi dengan arsitektur **Hybrid Offline-First**, di mana inferensi AI sepenuhnya on-device (Google Gemma via LiteRT-LM & ML Kit OCR) dan persistensi lokal berada di Room Database.

Namun, untuk mengoperasikan platform akademik yang handal, dibutuhkan **KerjaiAja Web Admin CMS** sebagai dashboard kontrol terpusat bagi administrator, tim dukungan pengguna, dan pengelola kurikulum akademik.

### Prinsip Utama CMS:
1. **Satu Sumber Data Kebenaran (Single Shared Backend):** Android dan Web CMS terhubung ke basis data Supabase PostgreSQL yang sama dengan batas isolasi hak akses (*Row Level Security*) yang tegas.
2. **Kedaulatan Privasi Mahasiswa (Zero Private Content Surveillance):** Administrator **DILARANG KERAS** melihat isi catatan pribadi, file foto soal, atau pesan percakapan rahasia pengguna. CMS hanya mengelola metadata akun, status operasional, laporan keluhan, dan referensi akademik umum.
3. **Bukan Server Inferensi AI (Zero Remote AI Inference):** CMS **BUKAN** AI backend. CMS tidak menyediakan endpoint inferensi, tidak menyimpan API key pihak ketiga, dan tidak menyimpan bobot model AI.
4. **Non-Invasive terhadap Offline-First Android:** CMS tidak dapat memanipulasi basis data lokal Room secara langsung. Setiap pembaruan referensi akademik masuk melalui alur sinkronisasi standar Supabase.

---

## 2. Identifikasi Masalah & Nilai Operasional Nyata

Mengapa KerjaiAja membutuhkan Web Admin CMS?

| Masalah Operasional Lapangan | Solusi Nyata via Web Admin CMS | Nilai Bisnis / Manfaat |
| :--- | :--- | :--- |
| **1. Pengelolaan Referensi & Silabus 12 Rumpun Mata Pelajaran** | Manajemen katalog kurikulum akademik (Matematika, Fisika, Biologi, dll.) agar pengguna Android mendapatkan referensi standar yang konsisten. | Konsistensi materi dan panduan kurikulum tanpa perlu rilis ulang APK. |
| **2. Penanganan Keluhan Pengguna & Laporan Bug** | Modul tiket keluhan, laporan ketidaksesuaian hasil OCR, dan feedback fitur dari pengguna Android. | Respons cepat terhadap bug dan peningkatan retensi pengguna mahasiswa. |
| **3. Moderasi Ruang Kerja Tim (Team Workspaces)** | Pemantauan status tim kelompok untuk mencegah penyalahgunaan nama kelompok yang melanggar etika akademik. | Lingkungan belajar yang aman dan mematuhi regulasi institusi pendidikan. |
| **4. Pengawasan Kuota Supabase Free Tier ($0)** | Pemantauan agregat ukuran basis data dan jumlah pengguna terdaftar untuk mencegah lonjakan kuota (*quota overrun*). | Menjaga kepatuhan biaya Rp0 tanpa risiko aplikasi terhenti (*paused*). |
| **5. Audit Jejak Administratif (Security Audit Trail)** | Pencatatan setiap aksi modifikasi data oleh admin secara otomatis dan *tamper-evident*. | Mencegah penyalahgunaan wewenang internal dan eskalasi hak istimewa (*privilege escalation*). |

---

## 3. Aktor & Pemetaan Peran (Role-Based Access Control)

CMS menerapkan prinsip hak istimewa terkecil (*Least Privilege*):

```text
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│   SUPER_ADMIN   │       │      ADMIN      │       │     SUPPORT     │
├─────────────────┤       ├─────────────────┤       ├─────────────────┤
│ • Full System   │       │ • Subject Mgmt  │       │ • View Feedback │
│ • Admin Mgmt    │       │ • Team Moderate │       │ • Respond Ticket│
│ • Audit Logs    │       │ • User Review   │       │ • Read Reference│
│ • Config Guard  │       │ • Feedback Act  │       │ • No Delete     │
└─────────────────┘       └─────────────────┘       └─────────────────┘
```

1. **`SUPER_ADMIN`:**
   - Mengelola akun staf administrator (invite, suspend, deprovisioning).
   - Membaca log audit keamanan sistem (`admin_audit_logs`).
   - Mengatur parameter konfigurasi platform dan ambang batas kuota.
2. **`ADMIN`:**
   - Mengelola katalog mata pelajaran dan referensi akademik (`academic_references`).
   - Memoderasi status ruang kerja tim yang dilaporkan melanggar ketentuan.
   - Mengelola status akun pengguna (misal: penonaktifan akun atas permintaan pengguna).
3. **`SUPPORT`:**
   - Meninjau dan merespon feedback pengguna serta laporan kesalahan OCR.
   - Membaca status kesehatan operasional umum.
   - *Read-only* pada sebagian besar data sensitif; dilarang menghapus data akun.

---

## 4. Klasifikasi Fitur CMS (MoSCoW Prioritization)

Berdasarkan analisis trade-off antara manfaat nyata, kompleksitas kode, dan batasan sumber daya:

### 4.1 MUST HAVE (Kritikal untuk MVP)
1. **CMS-01: Autentikasi Admin Terisolasi & Route Guarding:**
   - Login admin via Supabase Auth dengan email/password terdaftar.
   - Larangan mutlak registrasi publik (*No Public Self-Registration*).
   - Middleware Next.js untuk memproteksi seluruh rute `/admin/*` di sisi server.
2. **CMS-02: Dashboard Ringkasan Operasional (Executive Overview):**
   - Kartu metrik fungsional: Total Pengguna Terdaftar, Tim Aktif, Tiket Keluhan Terbuka, Estimasi Ukuran DB.
   - Grafik sederhana adopsi fitur (Mode A vs Mode B).
   - Status kuota Supabase Free Tier (indikator batas 500 MB DB dan 1 GB Storage).
3. **CMS-03: Manajemen Pengguna & Status Akun (User Management):**
   - Pencarian pengguna berdasarkan UID atau email anonim.
   - Filter status: Aktif, Ditangguhkan (*Suspended*), Ditandai Hapus (*Pending Deletion*).
   - Aksi administratif: Penangguhan akun atas pelanggaran atau pemrosesan hak hapus akun (*Right to Erasure* / UU PDP).
   - **Batasan Ketat:** Dilarang menampilkan password, token sesi, foto soal, atau catatan pribadi.
4. **CMS-04: Katalog Rumpun Mata Pelajaran & Referensi Akademik:**
   - CRUD data 12 rumpun mata pelajaran dan daftar topik standar kurikulum.
   - Status publikasi: Draft vs Published.
   - Terhubung langsung ke tabel `public.academic_references` yang dibaca oleh Android.
5. **CMS-05: Modul Umpan Balik & Laporan Bug (Feedback & Support):**
   - Daftar tiket feedback yang dikirimkan oleh pengguna Android.
   - Status tiket: `NEW`, `IN_REVIEW`, `RESOLVED`, `DISMISSED`.
   - Catatan internal admin (*admin resolution notes*).
6. **CMS-06: Audit Trail Logging (`admin_audit_logs`):**
   - Pencatatan otomatis setiap aksi mutasi data oleh admin (waktu UTC, admin UID, aksi, tabel target, record ID, IP hash).

### 4.2 SHOULD HAVE (Sprint Lanjutan)
1. **CMS-07: Moderasi Ruang Kerja Tim (Team Workspace Moderation):**
   - Peninjauan nama tim dan deskripsi kelompok yang dilaporkan pengguna.
   - Kemampuan membekukan tim (*freeze team*) jika melanggar ketentuan etika akademik.
2. **CMS-08: Manajemen Akun Administrator (Super Admin Only):**
   - Undangan admin baru via email terverifikasi (*secure invite token*).
   - Pengaturan role: `ADMIN` atau `SUPPORT`.

### 4.3 COULD HAVE (Masa Depan / Pasca-MVP)
1. **CMS-09: Registri Versi Prompt & Kebijakan AI:**
   - Registri metadata nomor versi prompt Asisten Kerja dan panduan format 7 bagian.
   - Klien Android mengecek versi metadata untuk memperbarui prompt template lokal secara berkala.

### 4.4 NOT IN MVP (Dilarang / Ditolak Secara Sadar)
1. **Remote AI Inference Engine:** Dilarang mengeksekusi LLM atau OCR di server Vercel/CMS.
2. **API Key Management untuk External AI:** Dilarang menyediakan form input OpenAI/Gemini/Anthropic API key.
3. **User Content Surveillance (Chat/Photo Spy):** Dilarang melihat berkas gambar foto soal atau percakapan catatan tugas pengguna.
4. **Direct Mutation ke Room SQLite:** Dilarang membypass arsitektur sinkronisasi Supabase.
5. **Vanity Metric Charts:** Dilarang menambahkan grafik animasi berat yang menghabiskan kuota egress Supabase.

---

## 5. Kriteria Penerimaan Produk (Acceptance Criteria)

1. **Keamanan & Otorisasi:**
   - Pengguna non-admin yang mengakses `/admin/*` langsung dialihkan ke `/auth/login` dengan status 403/Redirect.
   - Tidak ada token `service_role_key` yang terekspos di browser klien (wajib diperiksa via build check).
   - Semua mutasi data admin tercatat di tabel `admin_audit_logs`.
2. **Performa & Aksesibilitas:**
   - Halaman dashboard admin memiliki waktu muat *First Contentful Paint (FCP)* $\le 1.2\text{ detik}$ pada koneksi 4G.
   - Tabel data mengimplementasikan paginasi sisi server (*server-side pagination*) dengan batas maksimal 25 baris per halaman.
   - Antarmuka memenuhi standar kontras WCAG 2.1 AA dan responsif untuk layar desktop serta tablet.
3. **Integritas Sinkronisasi Android:**
   - Perubahan pada tabel referensi akademik di CMS langsung terbaca oleh Android saat sinkronisasi periodik tanpa merusak skema lokal Room.
