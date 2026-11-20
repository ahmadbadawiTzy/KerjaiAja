# ADR-014: On-Device AI Menggunakan Google Gemma & LiteRT-LM Runtime

## Status
**ACCEPTED** (2026-11-20) — Menggantikan ([SUPERSEDES](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/ARCHITECTURE_DECISIONS.md)) [ADR-004](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-004-BACKEND-AI-INFERENCE.md) dan [ADR-013](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-013-SELF-HOSTED-AI-LLAMACPP-QWEN.md).

---

## 1. Konteks (Context)
Sebelumnya, KerjaiAja merencanakan inferensi kecerdasan buatan melalui server mandiri (*self-hosted backend*) menggunakan model Qwen3 via daemon `llama.cpp` di atas VPS Linux. Meskipun strategi tersebut menjamin kedaulatan data dari API eksternal pihak ketiga, strategi tersebut menimbulkan beban biaya server bulanan ($5–$24/bulan), membutuhkan pengelolaan infrastruktur, dan bergantung pada koneksi internet untuk setiap interaksi analitik.

Di sisi lain, pengguna menginginkan aplikasi yang beroperasi **100% offline-first**, memiliki **biaya operasional Rp0**, dan menjamin **kerahasiaan data akademik mahasiswa secara mutlak** langsung pada perangkat ponsel.

---

## 2. Keputusan (Decision)
1. **Mengadopsi Google Gemma On-Device:**  
   KerjaiAja menetapkan keluarga model open-weight **Google Gemma (Gemma 4 E2B dan Gemma 3 1B)** sebagai fondasi kecerdasan buatan on-device.
2. **Runtime Resmi Google AI Edge LiteRT-LM:**  
   Menggunakan runtime resmi **LiteRT-LM** (sebelumnya TensorFlow Lite / MediaPipe GenAI) yang dioptimalkan untuk eksekusi LLM pada arsitektur CPU (NEON), GPU, dan NPU Android via Kotlin API.
3. **Pemberlakuan Multi-Token Prediction (MTP):**  
   Memanfaatkan kapabilitas MTP bawaan Gemma 4 pada LiteRT-LM untuk mempercepat *decode latency* hingga ~30–50% di perangkat bergerak.
4. **Non-Chat Contextual AI:**  
   AI dilarang keras diimplementasikan sebagai antarmuka chatbot terbuka atau general-purpose assistant. AI hanya diaktifkan secara transien untuk 5 tugas spesifik:
   - Intelligent Task Breakdown
   - Contextual Task Prioritization Analysis
   - Deadline Risk Detection
   - Smart Planning
   - Contextual Blocker Detection
5. **Dukungan Adaptif & Graceful Fallback:**  
   Jika perangkat tidak memenuhi syarat komputasi (RAM bebas < 1.5 GB atau baterai lemah), sistem beralih 100% ke Deterministic Productivity Engine dan template manual.

---

## 3. Konsekuensi (Consequences)
### Positif:
- **Biaya Operasional AI Rp0:** Tidak ada biaya sewa server, tidak ada biaya API per token.
- **Privasi Mutlak:** Nol byte data tugas atau percakapan pengguna yang meninggalkan ponsel.
- **Ketersediaan Offline:** Asistensi cerdas tetap berfungsi saat mahasiswa berada di ruang kuliah tanpa sinyal internet.
### Negatif:
- Memerlukan unduhan berkas bobot model (~1.1–1.6 GB) oleh pengguna.
- Perangkat kelas bawah (RAM $\le 4\text{ GB}$) tidak dapat menjalankan inferensi AI on-device (namun tetap dapat menggunakan seluruh fitur produktivitas non-AI secara normal).
