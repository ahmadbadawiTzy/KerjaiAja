# Catatan Arsitektur Backend (Backend Architecture Archive) — KerjaiAja

- **Status Dokumen:** **RETIRED & SUPERSEDED**
- **Tanggal Penghentian:** 2026-11-20
- **Dasar Keputusan:** [ADR-016: Removal of Go/Gin Backend & Zero Server Infrastructure](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-016-REMOVAL-OF-GO-GIN-BACKEND.md)
- **Arsitektur Pengganti:** [SYSTEM_ARCHITECTURE.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/SYSTEM_ARCHITECTURE.md) dan [ANDROID_ARCHITECTURE.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/ANDROID_ARCHITECTURE.md)

---

## 1. Ringkasan Penghentian Layanan Backend (Retirement Summary)

Pada rebaseline arsitektur fundamental (2026-11-20), tim arsitektur KerjaiAja secara sadar menghentikan dan menghapus seluruh rencana implementasi server backend Go/Gin beserta direktori scaffold `backend/`. 

Keputusan ini diambil setelah seluruh fungsionalitas inferensi kecerdasan buatan berhasil dialihkan ke **On-Device AI (Google Gemma via LiteRT-LM)** dan persistensi data dialihkan ke **Local-First (Room Database & DataStore)**. 

### Alasan Rekayasa Penghentian Backend:
1. **Peniadaan Biaya Server (Rp 0 Cost Policy):** Mengoperasikan server backend dan daemon AI membutuhkan sewa VPS bulanan ($5–$24/bulan). Dengan arsitektur on-device, biaya operasional server KerjaiAja menjadi **Rp 0 / $0**.
2. **Kedaulatan Privasi & Ketersediaan Offline:** Mahasiswa dapat menggunakan 100% fungsionalitas aplikasi di ruang kuliah tanpa koneksi internet, tanpa risiko downtime server atau keterlambatan jaringan.
3. **Penyederhanaan Basis Kode:** Menghilangkan overhead pemeliharaan multi-bahasa (Go + Kotlin) dan memusatkan seluruh rekayasa pada platform Android Native yang ramping, tangguh, dan mudah diuji.

---

## 2. Status Komponen Backend yang Telah Dihapus

| Komponen Backend Lama | Status Saat Ini | Alternatif Arsitektur Aktif |
| :--- | :---: | :--- |
| **Go Gin HTTP Server (`cmd/server/main.go`)** | ❌ **DIHAPUS** | Dieksekusi langsung di klien Android Native via Clean Architecture UseCases. |
| **Konfigurasi Lingkungan (`backend/.env`)** | ❌ **DIHAPUS** | Digantikan oleh DataStore Preferences dan Android Keystore lokal. |
| **Endpoint REST API (`/api/v1/...`)** | ❌ **RETIRED** | Digantikan oleh kontrak antarmuka internal Kotlin ([API_DESIGN.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/API_DESIGN.md)). |
| **Daemon Inferensi Server (`llama.cpp`)** | ❌ **DIHAPUS** | Digantikan oleh runtime **LiteRT-LM on-device** ([ADR-014](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-014-ON-DEVICE-GEMMA-LITERTLM.md)). |
| **Supabase PostgreSQL & Realtime Sync** | ❌ **DESCOPED** | Digantikan oleh **Room Database** lokal dan pertukaran berkas proyek kelompok `.kerjai` ([ADR-018](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-018-LOCAL-TEAM-WORKSPACE-PHASED-COLLABORATION.md)). |

*Dokumen ini disimpan dalam arsip dokumentasi teknis sebagai bukti jejak rekayasa (*engineering audit trail*) dan tidak lagi menjadi acuan implementasi aktif.*
