# ADR-025: Asisten Kerja Specification, Boundaries, and Ban on Generic Chatbots

## Status
Accepted

## Konteks
Fitur AI yang disajikan dalam bentuk generic conversational chatbot (seperti ChatGPT atau Gemini chat interface) sering kali membingungkan pengguna aplikasi produktivitas. Pengguna dihadapkan pada kotak teks kosong tanpa arah yang jelas, rentan terhadap halusinasi di luar ranah akademik, boros komputasi token, dan tidak terintegrasi langsung dengan struktur data tugas aplikasi.

## Keputusan
Kami menetapkan bahwa kapabilitas AI KerjaiAja berwujud **Asisten Kerja Kontekstual**, dengan aturan tegas:

### 1. Larangan Mutlak Terhadap Generic Chatbot
- Dilarang membuat layar chat obrolan bebas tanpa batas (*no open-ended chit-chat*).
- AI tidak bertindak sebagai asisten obrolan umum atau teman curhat.
- Dilarang menjawab pertanyaan di luar ranah manajemen akademik, perencanaan waktu, dan data tugas pengguna.

### 2. Lima Ranah Fitur Asisten Kerja
1. **Ask About My Work:** Pertanyaan terstruktur seputar beban tugas pengguna (misalnya: *"Apa saja tugas kritis yang harus selesai sebelum Jumat?"*, *"Berapa estimasi jam belajar yang saya butuhkan minggu ini?"*).
2. **Task Breakdown:** Memecah tugas besar menjadi 3–7 sub-tugas yang terukur, konkret, dan memiliki estimasi waktu realistis.
3. **Smart Planning:** Merekomendasikan jadwal blok belajar harian dengan mempertimbangkan jam kuliah/sekolah dan sisa energi mental.
4. **Contextual Recommendations:** Rekomendasi aksi tugas berikutnya berdasarkan deterministik skor prioritas dan konteks saat ini.
5. **Work Review:** Pengecekan pra-pengumpulan (apakah semua instruksi tugas dan lampiran penting sudah diceklis).

### 3. Output Terstruktur & Grounding Lokal (Local RAG)
- Semua output AI harus berupa JSON terstruktur atau Markdown terformat rapi sesuai skema data aplikasi.
- AI wajib di-grounding dengan konteks data tugas aktual pengguna yang diambil melalui SQLite FTS5 (Local RAG). Model dilarang mengarang tugas atau mata kuliah yang tidak ada di database lokal.

## Konsekuensi
### Positif
- Pengalaman pengguna menjadi sangat fokus, efisien, dan bernilai nyata untuk produktivitas.
- Menekan konsumsi token inferensi dan mempercepat waktu respons on-device.
- Mencegah halusinasi fakta dan memitigasi risiko keamanan prompt injection.

### Negatif / Mitigasi
- Pengguna yang mengharapkan AI untuk obrolan santai mungkin merasa terbatas (dimitigasi dengan penjelasan eksplisit mengenai peran Asisten Kerja di UI).
