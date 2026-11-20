# ADR-002: Go Backend with Gin Framework (Modular Monolith)

- **Status:** ACCEPTED
- **Tanggal:** 2026-11-20
- **Pengambil Keputusan:** Tim Arsitektur KerjaiAja (Go Backend Engineer)
- **Terkait Dokumen:** [`docs/architecture/BACKEND_ARCHITECTURE.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/BACKEND_ARCHITECTURE.md), [`docs/architecture/API_DESIGN.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/API_DESIGN.md)

---

## 1. Konteks (Context)
Untuk mendukung fitur Team Collaboration, Intelligent Task Prioritization, dan sinkronisasi lintas perangkat yang andal, KerjaiAja membutuhkan backend service yang cepat, hemat memori (*low resource footprint*), mudah di-maintain, dan dapat dijalankan secara lokal pada mesin pengembang dengan latensi minimal.

## 2. Keputusan (Decision)
Membangun backend service menggunakan bahasa **Go (Golang)** dengan web framework **Gin** dalam arsitektur **Modular Monolith**:
- **RESTful API Terversi:** Dimulai dari prefix `/api/v1`.
- **Domain-Driven Modular Package:** Struktur direktori di bawah `internal/` dikelompokkan berdasarkan domain bisnis: `auth`, `users`, `tasks`, `prioritization`, `teams`, `ai`, `notifications`, `audit`.
- **Database Access:** Menggunakan library native `pgx/v5` dan `sqlc` untuk kompilasi SQL type-safe berkinerja tinggi tanpa overhead ORM berat.
- **Middleware Terpusat:** Authentication (Supabase JWT verification), CORS, Rate Limiting, Request Logging (structured JSON), Panic Recovery, dan Team Role Authorization.
- **Konfigurasi:** Berbasis environment variables (`.env`) melalui library `godotenv`.
- **Pengujian:** Unit test handler dan service berbasis Go standard library `testing` dan `testify`.

## 3. Alternatif yang Dipertimbangkan (Alternatives Considered)
- **Node.js (Express / NestJS):** Ditolak karena konsumsi RAM lebih besar saat idle dan performa konkurensi I/O yang kalah efisien dibanding goroutines.
- **Microservices Architecture:** Ditolak keras untuk fase MVP; microservices menambah kompleksitas networking, tracing, dan deployment tanpa justifikasi beban pengguna.
- **Pure Go Standard Library (`net/http`):** Sangat baik, namun Gin dipilih karena routing radix tree yang sangat cepat, ekosistem middleware matang, dan binding/validation JSON deklaratif yang mempercepat siklus development.

## 4. Konsekuensi (Consequences)
- **Positif:** Eksekusi biner tunggal (*single static binary*), konsumsi RAM sangat rendah (< 30 MB idle), latensi sub-milidetik, mudah dijalankan secara lokal tanpa container berat.
- **Negatif:** Diperlukan disiplin ketat dalam pemisahan batas domain agar kode tidak menjadi "spaghetti monolith".
- **Implikasi Keamanan:** Backend bertindak sebagai *gatekeeper* utama; seluruh payload divalidasi dengan validator bawaan Gin; SQL injection dicegah secara total oleh parameterized queries `sqlc`/`pgx`.
- **Implikasi Biaya:** Zero cost untuk development lokal; siap di-deploy ke hosting gratis/murah (seperti Fly.io / Render / VPS 512MB RAM) di masa mendatang.
