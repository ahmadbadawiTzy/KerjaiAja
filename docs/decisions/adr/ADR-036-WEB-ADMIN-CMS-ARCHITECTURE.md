# ADR-036: Web Admin CMS Architecture and Technology Stack
## Arsitektur Dashboard Web Administrator Berbasis Next.js 15 App Router & Supabase SSR

- **Status:** APPROVED / LOCKED
- **Tanggal:** 2026-11-20
- **Penanggung Jawab:** Principal Software Architect, Senior Full-Stack Web Engineer
- **Konteks:** Platform KerjaiAja memerlukan dashboard web administratif untuk mengelola operasional kurikulum, menangani tiket feedback pengguna, dan memantau kesehatan sistem.

---

## 1. Konteks & Masalah

Pengelolaan operasional platform akademik KerjaiAja menuntut antarmuka web khusus bagi administrator yang aman, responsif, berkinerja tinggi, dan dapat diakses melalui peramban web modern tanpa menambah beban biaya server atau membuat duplikasi basis data.

Pilihan teknologi harus:
1. Kompatibel dengan Supabase Managed BaaS yang telah digunakan oleh aplikasi Android.
2. Dapat diterapkan dengan biaya operasional Rp0 ($0) memanfaatkan hosting Vercel (Hobby) atau platform kontainer mandiri (*self-hosted*).
3. Mendukung arsitektur *Server-First* untuk meminimalkan beban komputasi di sisi klien dan mengamankan kredensial server.
4. Memiliki ekosistem komponen antarmuka yang matang, mudah dipelihara, dan aksesibel.

---

## 2. Keputusan Arsitektur

Kami memutuskan untuk membangun **KerjaiAja Web Admin CMS** dengan spesifikasi teknologi dan arsitektur berikut:

1. **Framework Aplikasi Web:** **Next.js 15+ dengan App Router** dan **React 19**.
   - Memanfaatkan *React Server Components (RSC)* untuk mengambil data langsung di server.
   - Menggunakan *Server Actions* untuk mutasi data yang aman dengan validasi schema Zod.
2. **Bahasa & Pemrograman:** **TypeScript** dengan konfigurasi ketat (*Strict Mode*).
3. **Sistem Desain & Styling:** **Tailwind CSS** dipadukan dengan pustaka komponen **shadcn/ui** (berbasis Radix UI primitives) untuk menghasilkan antarmuka modern, bersih, dan mematuhi WAI-ARIA tanpa template generik yang berlebihan.
4. **Integrasi Autentikasi & Database:** **`@supabase/ssr`** via HTTP cookies aman (*HttpOnly, Secure, SameSite*).
5. **Manajemen Form & Validasi:** **Zod** untuk deklarasi skema validasi tipe data input dan *React Hook Form* untuk form kompleks.
6. **Testing Suite:** **Vitest** untuk pengujian unit/integrasi dan **Playwright** untuk pengujian *end-to-end (E2E)* smoke testing.
7. **Pemisahan Peran Tegas:** CMS **BUKAN** server inferensi AI dan tidak menyimpan API key pihak ketiga apa pun.

---

## 3. Konsekuensi

### Positif:
- **Performa Tinggi & Ukuran Bundle Ringan:** Server Components merender HTML di server, mengurangi JavaScript runtime yang harus diunduh browser admin.
- **Keamanan Kredensial:** Seluruh komunikasi istimewa ke Supabase dijalankan di lingkungan server, mencegah kebocoran secret key ke publik.
- **Portabilitas Tinggi:** Proyek dapat di-deploy ke Vercel atau dipaketkan ke dalam Docker container untuk self-hosted di cloud VPS gratis.
- **Satu Ekosistem Data:** Tidak ada backend tambahan yang perlu dirawat; menggunakan Supabase PostgreSQL yang sama dengan klien Android.

### Negatif / Mitigasi:
- **Keterbatasan Vercel Hobby:** Vercel Hobby membatasi penggunaan komersial. Mitigasi: Arsitektur dibuat 100% agnostik platform sehingga siap dipindahkan ke Docker/Coolify jika proyek bertransformasi komersial.

---

## 4. Referensi Dokumen Terkait

- [CMS Product Requirements](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/product/CMS_PRODUCT_REQUIREMENTS.md)
- [Web Admin Architecture](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/WEB_ADMIN_ARCHITECTURE.md)
- [Vercel & Supabase Deployment Strategy](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/deployment/VERCEL_SUPABASE_DEPLOYMENT.md)
- [CMS Test Strategy](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/testing/CMS_TEST_STRATEGY.md)
