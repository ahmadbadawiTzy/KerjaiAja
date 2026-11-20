# ADR-004: Backend-Based AI Inference Architecture

- **Status:** ACCEPTED (Menggantikan Keputusan On-Device Gemma 3 1B IT / LiteRT-LM, Ditegaskan & Diperketat oleh [ADR-013: Self-Hosted AI Architecture](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-013-SELF-HOSTED-AI-LLAMACPP-QWEN.md))
- **Tanggal:** 2026-11-20
- **Pengambil Keputusan:** Tim Arsitektur KerjaiAja (AI/ML Architect & Senior Platform Architect)
- **Terkait Dokumen:** [`docs/ai/AI_ARCHITECTURE.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/ai/AI_ARCHITECTURE.md), [`docs/ai/TASK_PRIORITIZATION_SPEC.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/ai/TASK_PRIORITIZATION_SPEC.md), [`docs/decisions/adr/ADR-013-SELF-HOSTED-AI-LLAMACPP-QWEN.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-013-SELF-HOSTED-AI-LLAMACPP-QWEN.md)

---

## 1. Konteks (Context)
Pada perancangan awal, KerjaiAja merencanakan inferensi on-device menggunakan Gemma 3 1B IT via LiteRT-LM. Berdasarkan audit mendalam (`docs/audits/SUPERPOWERS_DOCUMENTATION_AUDIT.md`), inferensi on-device pada perangkat Android kelas menengah (RAM 4–6 GB) terbukti memiliki risiko fatal:
1. Lonjakan alokasi memori runtime (800 MB – 1.2 GB) memicu pemutusan paksa oleh Android *Low Memory Killer Daemon* (LMKD).
2. Keterbatasan kapasitas penalaran model 1B INT4 dalam menghasilkan keluaran JSON terstruktur dan Bahasa Indonesia konteks akademik yang bernuansa.
3. Kebutuhan pengunduhan model biner on-demand (~600–900 MB) yang membebani kuota data seluler pelajar.

Selain itu, KerjaiAja kini memperluas kemampuannya mencakup **Intelligent Task Prioritization** dan **Team Collaboration**, yang membutuhkan reasoning lintas konteks tugas yang tidak mungkin dijalankan dengan stabil di edge device entry-level.

## 2. Keputusan (Decision)
Memindahkan seluruh eksekusi **AI Inference ke Backend (Backend-Based AI Inference)** dengan prinsip **100% Self-Hosted**:
1. **Orkestrasi Backend Go:**
   - Go Backend bertindak sebagai orkestrator inferensi AI (`internal/ai`): mengelola concurrency limits via semaphore, menyusun prompt terstruktur, memanggil internal inference service, memvalidasi skema JSON, dan menangani fallback deterministik.
2. **Kunci Model & Runtime Self-Hosted (Terkunci via ADR-013):**
   - **Model Utama:** **Qwen3-4B GGUF Q4_K_M** (dengan **Qwen3-1.7B** sebagai fallback eksperimental resource minim).
   - **Inference Engine:** **`llama.cpp`** yang berjalan sebagai private service lokal (unexposed ke publik).
3. **Larangan Mutlak Penggunaan AI API Eksternal:**
   - *Catatan Revisi Arsitektur:* Opsi penggunaan cloud AI API pihak ketiga (Gemini, OpenAI, Anthropic, Groq, Hugging Face Inference API) secara resmi **DIBATALKAN & DILARANG**. Seluruh inferensi wajib dieksekusi secara mandiri pada infrastruktur server sendiri tanpa mengirim data pengguna ke pihak ketiga mana pun.
4. **Structured Response Enforcement & Bounded Timeout:**
   - Go backend menerapkan timeout ketat (10 detik) dan validasi skema JSON. Jika model internal timeout atau error, sistem segera beralih ke template deterministik tanpa mengorbankan pengalaman pengguna.
5. **Batas Konteks & Izin Data (Permission Boundaries):**
   - Backend membatasi konteks AI secara ketat: pengguna hanya dapat melakukan query terhadap tugas miliknya sendiri dan tugas tim tempat pengguna menjadi anggota aktif terverifikasi.

## 3. Alternatif yang Dipertimbangkan (Alternatives Considered)
- **Tetap On-Device (Gemma 3 1B IT LiteRT-LM):** Ditolak secara sadar karena membatasi audiens pengguna smartphone 4 GB RAM, menghabiskan memori lokal, dan menyulitkan pembaruan model secara terpusat.
- **External Cloud AI APIs (Gemini/OpenAI/Anthropic):** Ditolak mutlak karena risiko privasi data akademik mahasiswa, ketidakstabilan biaya token jangka panjang, dan ketergantungan pada vendor pihak ketiga.
- **Hybrid Edge-Cloud:** Ditolak untuk fase MVP demi menghindari kompleksitas ganda dalam memelihara dua runtime AI yang berbeda.

## 4. Konsekuensi (Consequences)
- **Positif:**
  - Klien Android menjadi sangat ringan (< 15 MB APK, < 80 MB RAM base).
  - 100% kompatibel dengan seluruh smartphone Android (bahkan ponsel RAM 2–3 GB sekalipun).
  - Privasi data pengguna 100% berada di bawah kendali sendiri tanpa ada data yang bocor ke penyedia AI komersial.
  - Kualitas penalaran jauh lebih tinggi dengan model Qwen3-4B dalam memahami instruksi akademik Bahasa Indonesia.
- **Negatif / Mitigasi:**
  - Membutuhkan alokasi resource komputasi mandiri pada server (RAM/CPU/GPU) untuk menjalankan binari `llama.cpp`. Diminimalkan dengan kuantisasi efisien `Q4_K_M`.
