# ADR-033: Progressive AI Processing Pipeline (Deterministic-First Architecture)

## Status
ACCEPTED (Disetujui) — 2026-11-20

## Konteks
Dalam arsitektur komputasi perangkat bergerak Android, menjalankan model bahasa besar (*Large Language Model*) atau *Vision Transformer* merupakan operasi yang sangat membebani sumber daya (memori RAM $> 1\text{ GB}$, beban komputasi CPU/NPU masif, dan pengurasan daya baterai). Pada aplikasi produktivitas pelajar yang beroperasi secara offline dan gratis, mendelegasikan seluruh tugas sederhana (seperti perhitungan deadline, pemilahan prioritas, validasi input, atau ekstraksi teks) langsung ke LLM menimbulkan inefisiensi ekstrem, latensi tinggi, dan risiko crash akibat *LowMemoryKiller (LMK)*.

Diperlukan arsitektur pemrosesan bertingkat yang memprioritaskan pemrosesan deterministik murni sebelum memanggil model AI.

## Keputusan
Menetapkan arsitektur **Progressive AI Processing Architecture** dengan prinsip:  
*"Do as much as possible with lightweight deterministic processing before invoking an AI model."*

Pipeline dibagi ke dalam 7 tahapan sekuensial yang terdefinisi ketat:
1. **Stage 0 (Native Deterministic Processing):** Menyelesaikan tugas-tugas terstruktur (perhitungan deadline, durasi, rumus prioritas 0–100, validasi form, sorting) murni dengan kode Kotlin ($< 5\text{ ms}$, 0 byte RAM LLM).
2. **Stage 1 (Lightweight Input Preprocessing):** Sanitasi teks, validasi MIME citra, subsampling bitmap proporsional ($\le 1920\times 1080$), auto-orientasi EXIF, dan deteksi blur Laplacian sebelum alokasi memori berlebih.
3. **Stage 2 (Task Classification & Intelligent Routing):** Penentuan rute pemrosesan berbasis aturan (*rule-based router*) deterministik (Zero-AI Path vs OCR-First Path vs Text Assistant Path) tanpa memanggil classifier LLM.
4. **Stage 3 (OCR-First Processing):** Ekstraksi teks citra menggunakan Google ML Kit Text Recognition v2 ($< 35\text{ MB}$ RAM) dilengkapi layar peninjauan dan koreksi pengguna (*Human-in-the-Loop Review*) dengan toolbar simbol akademik.
5. **Stage 4 (Local Retrieval & Context Optimization):** Ekstraksi konteks relevan via SQLite FTS5 BM25 dan pembatasan token budget ketat ($\le 800\text{ token}$).
6. **Stage 5 (AI Model Inference):** Inferensi terisolasi on-device via LiteRT-LM (Gemma 3 1B IT / 270M) dengan sesi tunggal (*single active session*) dan pengaman termal.
7. **Stage 6 (Post-processing & Adaptive Subject Validation):** Validasi kelengkapan 7 bagian terstruktur yang adaptif terhadap 12 rumpun mata pelajaran, verifikasi aritmetika deterministik, dan pembersihan berkas citra efemeral.

## Konsekuensi
- **Positif:**
  - Pengurangan konsumsi RAM puncak dan eliminasi 100% risiko OOM crash pada tugas manajerial harian.
  - Latensi respons aplikasi produktivitas instan ($< 5\text{ ms}$) untuk 100% pengguna.
  - Penghematan konsumsi daya baterai hingga $65\% - 75\%$ dibanding pendekatan konvensional (all-to-LLM).
  - Verifikasi aritmetika deterministik mencegah halusinasi kalkulasi matematika.
  - Aplikasi 100% fungsional dan bermanfaat tanpa perlu mengunduh model AI.
- **Negatif / Tantangan:**
  - Diperlukan pemeliharaan logika router berbasis aturan dan validator deterministik di kode Kotlin klien.
  - Soal visual kompleks (diagram geometri/grafik sains) memerlukan input teks tambahan dari siswa pada perangkat standar jika model vision Jalur B belum diunduh.

## Dokumen Terkait
- [PROGRESSIVE_AI_PROCESSING.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/PROGRESSIVE_AI_PROCESSING.md)
- [ADR-027-PHOTO-QUESTION-SOLVER-SPECIFICATION.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-027-PHOTO-QUESTION-SOLVER-SPECIFICATION.md)
- [ADR-028-OCR-FIRST-QUESTION-PROCESSING.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-028-OCR-FIRST-QUESTION-PROCESSING.md)
- [ADR-031-ACADEMIC-ANSWER-VALIDATION-AND-QUALITY-CONTROL.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-031-ACADEMIC-ANSWER-VALIDATION-AND-QUALITY-CONTROL.md)
