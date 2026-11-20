# Katalog Fitur Produk (Feature Catalog) — KerjaiAja
## Pemetaan Modul Fungsional Mode A (Tanpa AI) & Mode B (Dengan AI On-Device)

Dokumen ini menyajikan katalog lengkap fitur aplikasi **KerjaiAja**, batasan dependensi, dan ketersediaannya pada Mode A dan Mode B.

---

## 1. Matriks Fitur Global

| ID | Nama Fitur | Deskripsi Singkat | Mode A (Tanpa AI) | Mode B (Dengan AI) | Kebutuhan Jaringan |
| :--- | :--- | :--- | :---: | :---: | :---: |
| **F-01** | **Task & Subtask CRUD** | Pengelolaan tugas akademik dan checklist langkah mandiri | ✅ Ya | ✅ Ya | 100% Offline |
| **F-02** | **Subject & Schedule Hub**| Pengelompokan tugas per mata kuliah & jadwal kuliah mingguan | ✅ Ya | ✅ Ya | 100% Offline |
| **F-03** | **Deterministic Priority** | Perhitungan skor urgensi matematis transparan (0–100) | ✅ Ya | ✅ Ya | 100% Offline |
| **F-04** | **Anti-Guilt UX Flow** | Aksi pemulihan adaptif saat tugas overdue (*reschedule / split*) | ✅ Ya | ✅ Ya | 100% Offline |
| **F-05** | **Focus Pomodoro Timer** | Timer belajar terikat langsung ke sub-tugas aktif | ✅ Ya | ✅ Ya | 100% Offline |
| **F-06** | **Pre-Flight Submission** | Checklist verifikasi format berkas sebelum pengumpulan | ✅ Ya | ✅ Ya | 100% Offline |
| **F-07** | **Offline-First Sync** | Sinkronisasi background Room ke Supabase PostgREST | ✅ Ya (Jika Login) | ✅ Ya (Jika Login) | Online Saat Sync |
| **F-08** | **Team Collaboration** | Ruang kerja kelompok real-time via Supabase Realtime & RLS | ✅ Ya (Jika Login) | ✅ Ya (Jika Login) | Online |
| **F-09** | **Device Capability Check**| Evaluasi multi-faktor RAM, ABI, Storage, Thermal, Baterai | ✅ Ya | ✅ Ya | 100% Offline |
| **F-10** | **Asisten: Ask About Work**| Tanya status dan beban tugas terdekat berbasis fakta FTS5 | ❌ Tidak | ✅ Ya | 100% Offline |
| **F-11** | **Asisten: Task Breakdown**| Dekomposisi tugas besar menjadi 3–7 sub-tugas terstruktur | ❌ Tidak | ✅ Ya | 100% Offline |
| **F-12** | **Asisten: Smart Planning**| Saran alokasi blok waktu belajar adaptif | ❌ Tidak | ✅ Ya | 100% Offline |
| **F-13** | **Asisten: Work Review** | Pemeriksaan kesiapan checklist tugas pra-pengumpulan | ❌ Tidak | ✅ Ya | 100% Offline |
| **F-14** | **Foto Soal: Jalur A (OCR)**| Pemotretan soal teks, ekstraksi ML Kit, pembahasan Gemma | ❌ Tidak | ✅ Ya | 100% Offline |
| **F-15** | **Foto Soal: Jalur B (Vision)**| Penalaran visual langsung pada diagram geometri/grafik STEM | ❌ Tidak | 🔬 Evaluasi Lab | 100% Offline |
| **F-16** | **Answer Validator** | Validasi deterministik aritmetika, rumus, dan konsistensi | ❌ Tidak | ✅ Ya | 100% Offline |
| **F-17** | **Local Learning Notes RAG**| Injeksi catatan belajar lokal ke prompt pembahasan soal | ❌ Tidak | ✅ Ya | 100% Offline |

---

## 2. Batasan Keras & Anti-Features (Fitur yang DILARANG)
1. ❌ **Generic Chit-Chat Screen:** Dilarang membuat layar obrolan bebas tanpa arah.
2. ❌ **Auto Cloud Upload Foto:** Dilarang mengunggah foto soal ke cloud tanpa izin eksplisit.
3. ❌ **External AI API Calls:** Dilarang memanggil Gemini API, OpenAI API, Anthropic, Groq, dll.
4. ❌ **Forced AI Lock-in:** Dilarang memblokir fitur produktivitas inti bagi pengguna tanpa model AI.
