# SPESIFIKASI ASISTEN KERJA & BATASAN SISTEM (AI ASSISTANT SPECIFICATION) — KERJAIAJA
## Task-Contextual Intelligence · Zero Generic Chatbot · Strict Local Grounding

**Dokumen:** `docs/ai/AI_ASSISTANT_SPECIFICATION.md`  
**Status:** Baseline Disetujui  
**Model Runtime:** Google Gemma on-device via LiteRT-LM  
**Retriever:** Local RAG via SQLite FTS5 BM25  

---

## 1. Filosofi Desain: Asisten Kerja vs. Chatbot Generik

KerjaiAja dirancang bukan untuk menjadi aplikasi obrolan umum, melainkan sebuah **Asisten Kerja Kontekstual Akademik**. 

### Perbedaan Mendasar:
- **Chatbot Generik (DILARANG):** Kotak input kosong tak terbatas yang melayani obrolan umum (puisi, lelucon, obrolan politik, kode acak). Rentan halusinasi, boros memori, dan mengaburkan esensi produktivitas.
- **Asisten Kerja KerjaiAja (DISETUJUI):** Antarmuka terarah yang langsung berorientasi pada data tugas, jadwal, dan beban kerja pengguna saat ini. Setiap interaksi memiliki tujuan produktivitas yang konkret dan terukur.

---

## 2. Lima Use Case Inti Asisten Kerja

### 1. Ask About My Work (Tanya Tugas & Beban Kerjaku)
- **Tujuan:** Memberikan jawaban cepat dan terarah mengenai status beban kerja akademik pengguna.
- **Contoh Kueri:**
  - *"Apa saja tugas kritis yang harus dikumpulkan dalam 48 jam ke depan?"*
  - *"Berapa perkiraan total jam belajar yang saya butuhkan untuk menyelesaikan tugas Kalkulus minggu ini?"*
  - *"Mata kuliah apa yang memiliki beban tugas paling berat saat ini?"*
- **Mekanisme Grounding:** Kueri pengguna memicu `LocalRagRetriever` untuk mencari tugas relevan di Room FTS5. Potongan tugas (judul, deadline, estimasi waktu, status) diinjeksi ke dalam sistem prompt Gemma sebagai konteks fakta.

### 2. Task Breakdown (Dekomposisi Tugas Otomatis)
- **Tujuan:** Mengubah tugas besar yang menakutkan menjadi 3–7 langkah sub-tugas yang terukur, realistis, dan berurutan.
- **Input:** Judul tugas, deskripsi instruksi dari dosen, dan estimasi total durasi.
- **Output AI:** Array sub-tugas JSON terstruktur dengan estimasi waktu per langkah.
  ```json
  {
    "subtasks": [
      { "title": "Cari 3 jurnal referensi tentang Algoritma Dijkstra", "estimated_minutes": 45 },
      { "title": "Buat kerangka laporan & pseudocode algoritma", "estimated_minutes": 60 },
      { "title": "Implementasikan kode dalam Python & simulasikan graf", "estimated_minutes": 90 },
      { "title": "Susun bab analisis kompleksitas waktu & review akhir", "estimated_minutes": 45 }
    ]
  }
  ```
- Pengguna dapat langsung menekan tombol *"Terapkan Sub-Tugas"* untuk memasukkannya ke database Room dengan 1 ketukan.

### 3. Smart Planning (Rekomendasi Alokasi Waktu Belajar)
- **Tujuan:** Menyarankan alokasi sesi fokus (blok waktu belajar) berdasarkan tenggat waktu terdekat, estimasi kesulitan, dan jadwal luang di kalender akademik pengguna.
- **Prinsip Anti-Guilt:** Tidak memberikan jadwal yang tidak realistis (seperti belajar 8 jam non-stop). Menyelipkan jeda istirahat dan fleksibilitas.

### 4. Contextual Recommendations (Saran Tindakan Prioritas)
- **Tujuan:** Menjawab kebingungan umum pelajar: *"Sekarang saya harus mengerjakan apa?"*.
- **Mekanisme:** Menggabungkan deterministik score dari `DeterministicPriorityEngine` (faktor deadline, kesulitan, bobot) dengan penjelasan naratif singkat mengapa tugas tersebut perlu dikerjakan sekarang (misalnya: *"Mulai Tugas Fisika Dasar sekarang karena memiliki estimasi waktu 3 jam dan tenggat besok siang"*).

### 5. Work Review (Pengecekan Kesiapan Pra-Pengumpulan)
- **Tujuan:** Memeriksa kesiapan tugas sebelum dikumpulkan ke portal akademik/dosen.
- **Mekanisme:** Mengevaluasi apakah seluruh sub-tugas checklist telah tercentang, apakah format penamaan berkas sudah sesuai instruksi di deskripsi tugas, dan memunculkan ringkasan konfirmasi pengumpulan.

---

## 3. Batasan Sistem & Guardrails Prompting

Setiap prompt yang dikirim ke model on-device Gemma dibungkus dalam template sistem yang ketat:

```text
[INST]
Peran: Anda adalah Asisten Kerja akademik KerjaiAja. Anda bertugas membantu pelajar mengelola tugas secara efisien, terstruktur, dan realistis.

Batasan Wajib:
1. Jawab HANYA berdasarkan konteks tugas yang disediakan di bawah ini.
2. JANGAN mengarang tugas, mata kuliah, atau tenggat waktu yang tidak tercantum dalam konteks.
3. JANGAN melayani obrolan umum, hiburan, atau topik di luar manajemen tugas akademik. Jika pengguna bertanya di luar topik, tolak secara sopan dan arahkan kembali ke tugas akademik.
4. Jawab dalam Bahasa Indonesia yang ringkas, jelas, dan ramah.

Konteks Tugas Pengguna:
{context_chunks}

Pertanyaan Pengguna:
{user_query}
[/INST]
```

---

## 4. Evaluasi Kualitas & Keandalan Output

Kualitas Asisten Kerja dievaluasi secara berkala dengan parameter:
1. **JSON Parsing Success Rate:** $\ge 98\%$ keberhasilan deserialisasi output ke data class Kotlin.
2. **Context Adherence:** 0% halusinasi pada data tugas yang tidak ada di konteks FTS5.
3. **Response Latency:** Waktu hingga token pertama (*Time-to-First-Token / TTFT*) $\le 1.5$ detik pada perangkat kategori `RECOMMENDED`.
