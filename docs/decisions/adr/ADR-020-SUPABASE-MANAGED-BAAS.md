# ADR-020: Supabase Managed BaaS Integration via Official Kotlin Client

## Status
Accepted

## Konteks
KerjaiAja membutuhkan backend service untuk autentikasi pengguna, penyimpanan data cloud terpusat, sinkronisasi lintas perangkat, dan kolaborasi tim untuk proyek akademik. Membangun dan merawat custom backend seperti Go/Gin, Node.js, atau Python membutuhkan infrastruktur server mandiri (VPS), beban maintenance yang tinggi, konfigurasi DevOps berlebih, serta potensi biaya operasional rutin.

## Keputusan
Kami memutuskan untuk menggunakan **Supabase** sebagai Backend-as-a-Service (BaaS) terkelola secara langsung dari aplikasi Android Native melalui pustaka resmi **Supabase Kotlin Client (supabase-kt)**:
- Versi BOM: `io.github.jan-tennert.supabase:bom:3.9.0`
- HTTP Engine: Ktor Android Client (`io.ktor:ktor-client-android:3.0.3`)
- Modul yang digunakan: `auth-kt`, `postgrest-kt`, `realtime-kt`, dan `storage-kt`.

### Batasan Keras & Larangan
1. **Dilarang Membangun Custom Backend:** Tidak ada backend custom Go, Gin, Node.js, atau Python yang dibangun atau di-deploy.
2. **Dilarang Menyimpan AI Weights di Supabase Storage:** Model weights on-device Gemma (.bin/.litertlm) tidak boleh diunggah atau disimpan di Supabase Storage untuk menghemat kuota bandwidth dan storage Free Tier.
3. **Dilarang Inference di Cloud:** Supabase Edge Functions atau Database Webhooks tidak digunakan untuk inference model AI apa pun.

## Konsekuensi
### Positif
- Beban pemeliharaan server berkurang drastis menjadi nol (zero infrastructure management).
- Keamanan data multi-tenant dijamin langsung di level database via PostgreSQL Row Level Security (RLS).
- Mendukung sinkronisasi real-time instan melalui fitur CDC (Change Data Capture) Supabase Realtime.
- Beroperasi penuh dalam kuota Supabase Free Tier tanpa biaya hosting bulanan.

### Negatif / Mitigasi
- Ketergantungan pada konektivitas jaringan untuk sync ke cloud (dimitigasi dengan pendekatan offline-first menggunakan Room Database).
- Batas kuota Free Tier (dimitigasi dengan aturan guardrails kuota: 500 MB DB, 1 GB Storage, 2 GB/bulan egress).
