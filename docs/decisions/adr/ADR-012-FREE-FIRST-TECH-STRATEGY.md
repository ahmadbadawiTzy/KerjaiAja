# ADR-012: Free-First Technology Strategy, Open-Weights AI & Cost Policy

## Status
ACCEPTED (2026-11-20, Diperbarui untuk Menyelaraskan dengan [ADR-013](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-013-SELF-HOSTED-AI-LLAMACPP-QWEN.md))

## Konteks
Sebagai inisiatif produk independen dan open-collaboration, proyek KerjaiAja harus dapat dikembangkan, diuji, dan dijalankan dengan biaya awal nol (*zero upfront software license cost*) tanpa mengorbankan kualitas arsitektur, stabilitas teknis, privasi pengguna, maupun keamanan.

Namun, terdapat pemahaman keliru yang perlu ditegaskan secara transparan dalam kebijakan biaya (*Cost Policy*): tidak ada arsitektur produksi yang benar-benar "bebas biaya server tanpa batas". Penggunaan model AI open-weights dan self-hosted inferensi mengeliminasi biaya lisensi perangkat lunak dan biaya API pihak ketiga (*per-token fees*), namun infrastruktur komputasi fisik (CPU/RAM/GPU, storage, bandwidth) tetap membutuhkan alokasi sumber daya yang nyata.

## Keputusan
KerjaiAja menetapkan prinsip **Free-First Technology Strategy, Open-Weights Model, & Transparent Cost Policy**:
1. **Pemanfaatan Ekosistem Open Source Mandiri:**
   - **Language & Runtime:** Go 1.22+ (sangat hemat memori/CPU, binari tunggal tanpa overhead JVM/Node.js di server) dan Kotlin (native Android).
   - **Framework & Libraries:** Gin, pgx, sqlc, Jetpack Compose, Room, llama.cpp—semuanya open-source dengan lisensi permisif (MIT, Apache 2.0, BSD) tanpa biaya lisensi komersial.
2. **Kemandirian AI dengan Model Open-Weight (Zero AI API Cost):**
   - Menggunakan model **open-weight Qwen3-4B (GGUF Q4_K_M)** yang dijalankan secara mandiri via runtime **`llama.cpp`**.
   - *Pembaruan Aturan:* Penggunaan API cloud pihak ketiga (Gemini API, OpenAI, Anthropic, Groq) **DIBATALKAN & DILARANG**. Sistem tidak mengeluarkan biaya API eksternal per-token sama sekali.
3. **Managed Cloud Free Tiers untuk Fase Bootstrapping:**
   - **Database & Auth:** Supabase Free Tier menyediakan PostgreSQL 500MB, Auth hingga 50.000 Monthly Active Users (MAU), 1GB File Storage, dan Realtime connections.
   - **Local Development Parity:** Dukungan penuh Docker & Supabase CLI lokal (`supabase start`), memungkinkan seluruh stack backend dan database berjalan secara offline dan gratis di mesin developer tanpa perlu terhubung ke cloud.
   - **Version Control & CI:** GitHub Free Actions & GitHub Packages.
4. **Kebijakan Pemisahan Kategori Biaya (Transparent Cost Breakdown):**
   - **Biaya Model & Lisensi ($0):** Menggunakan model open-weight Qwen3-4B yang bebas biaya royalti.
   - **Biaya API Inferensi ($0):** Bebas dari biaya tagihan API pihak ketiga per-token.
   - **Biaya Komputasi Server (Realistis):** Membutuhkan server VPS/Compute dengan spesifikasi minimal 4–8 GB RAM untuk menjalankan container Go dan `llama.cpp` CPU inference, atau cloud spot instance GPU untuk throughput tinggi saat produksi.
   - **Biaya Penyimpanan (Storage):** Membutuhkan ~3–4 GB disk space untuk menyimpan berkas bobot model GGUF dan database.
   - **Biaya Bandwidth Jaringan:** Efisien karena hanya mengirimkan payload teks JSON terkompresi antara klien Android dan backend Go.

## Konsekuensi
### Positif
- Biaya lisensi software $0 selama fase perancangan, pengembangan, dan pengujian MVP.
- Mahasiswa dan developer lain dapat melakukan *clone* repository dan menjalankan seluruh ekosistem di komputer lokal secara offline tanpa biaya kartu kredit.
- Tidak ada risiko tagihan tak terduga (*bill shock*) akibat lonjakan pemanggilan API AI eksternal.

### Negatif / Mitigasi
- Server hosting untuk self-hosted inferensi memerlukan spesifikasi RAM/CPU yang lebih tinggi daripada server web biasa.
  - *Mitigasi:* Menggunakan kuantisasi efisien `Q4_K_M`, concurrency limiter pada Go backend, dan mempersiapkan model Qwen3-1.7B untuk server berspesifikasi hemat.
