# ADR-016: Penghapusan Backend Go/Gin & Eliminasi Infrastruktur Server

## Status
**ACCEPTED** (2026-11-20) — Menggantikan ([SUPERSEDES](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/ARCHITECTURE_DECISIONS.md)) [ADR-002](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-002-GO-GIN-MODULAR-MONOLITH.md).

---

## 1. Konteks (Context)
Pada rebaseline sebelumnya, KerjaiAja merencanakan backend Go Gin sebagai *modular monolith* untuk melayani endpoint REST `/api/v1` dan mengorkestrasi inferensi LLM di private server. Namun, dengan dipindahkannya seluruh fungsionalitas inferensi ke on-device (Gemma via LiteRT-LM) dan adopsi arsitektur local-first, server perantara tidak lagi memiliki tanggung jawab komputasi kritis untuk fungsionalitas inti MVP.

Mempertahankan server backend menimbulkan biaya sewa VPS, beban pemeliharaan DevOps, serta memperluas area kerentanan keamanan tanpa memberikan nilai tambah langsung bagi mahasiswa yang menggunakan aplikasi secara offline.

---

## 2. Keputusan (Decision)
1. **Penghapusan Total Rencana & Scaffold Go/Gin:**  
   Menghapus seluruh direktori `backend/`, binari server, berkas konfigurasi lingkungan, dan skrip pengujian backend Go dari repositori KerjaiAja.
2. **Tidak Mengganti dengan Backend Lain:**  
   Proyek **DILARANG** mengganti Go Gin dengan Node.js, Python FastAPI, Express, atau runtime backend server lainnya untuk MVP v1.0.
3. **Pemberlakuan Arsitektur Zero-Server Infrastructure:**  
   KerjaiAja beroperasi sebagai aplikasi Android Native mandiri (*standalone client*) yang seluruh logika bisnis, kalkulasi prioritas, dan penyimpanan datanya berada pada perangkat ponsel pengguna.

---

## 3. Konsekuensi (Consequences)
### Positif:
- **Biaya Pemeliharaan Server Rp0:** Menghilangkan biaya operasional bulanan secara permanen.
- **Sederhana & Mudah Dipelihara:** Pengembang hanya fokus pada satu basis kode (Kotlin / Android Native).
- **Surface Area Serangan Nol:** Tidak ada server web atau port publik yang rentan terhadap serangan DDoS, injeksi SQL eksternal, atau kebocoran kredensial server.
### Negatif:
- Fungsionalitas kolaborasi multi-perangkat yang bergantung pada server terpusat harus dirancang ulang menggunakan mekanisme lokal (ekspor/impor berkas).
