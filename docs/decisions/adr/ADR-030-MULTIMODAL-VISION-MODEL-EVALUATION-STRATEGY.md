# ADR-030: Strategi Evaluasi Model Vision Multimodal (Multimodal Vision Model Evaluation Strategy)

## Status
Accepted

## Konteks
Sebagian soal akademik dalam bidang sains, teknologi, rekayasa, dan matematika (STEM) serta geografi melibatkan representasi informasi visual murni yang tidak dapat diubah menjadi teks biasa oleh OCR, seperti:
- Diagram geometri bidang dan ruang (lingkaran singgung, sudut segitiga).
- Grafik fungsi matematika, kurva termodinamika, diagram fase.
- Gambar sel biologi, struktur anatomi, diagram persilangan genetika.
- Rangkaian listrik Kirchoff dan skema gaya bebas fisika Newton.

Google AI Edge LiteRT-LM mendukung integrasi model multimodal visual on-device. Google Gemma memperkenalkan varian multimodal generasi terbaru (seperti Gemma 3n E2B). Namun, model multimodal memiliki beban komputasi dan memori yang sangat masif, serta risiko halusinasi visual yang harus diuji secara empiris sebelum dinyatakan siap rilis produksi.

## Keputusan
Kami menetapkan **Strategi Evaluasi Model Vision Multimodal**:

### 1. Status Gemma 3n E2B sebagai Kandidat Evaluasi (Not Default)
- **Gemma 3n E2B** ditetapkan sebagai **Kandidat Evaluasi Resmi Jalur B**, BUKAN model default aplikasi.
- Model ini tidak akan diaktifkan secara default untuk seluruh pengguna. Model hanya ditawarkan kepada pengguna dengan perangkat kualifikasi tingkat tinggi (*Tier RECOMMENDED Vision*).

### 2. Parameter Evaluasi Komparatif Fisik di Lab Smartphone
Sebelum Gemma 3n E2B diizinkan melayani produksi, tim engineering wajib mengukur metrik berikut pada smartphone fisik:
1. **Ukuran Berkas Terkuantisasi (INT4):** Target $\le 1.8\text{ GB}$.
2. **Konsumsi Puncak RAM Runtime:** Target $\le 2.2\text{ GB}$ (dengan KV cache dan image encoder buffers aktif).
3. **Inference Latency (Image + Prompt):** Target Time-to-First-Token $\le 3.0$ detik pada GPU mobile.
4. **Thermal Impact & Battery Drain:** Kenaikan suhu $\le 3.5^\circ\text{C}$ per sesi 3 pertanyaan berturut-turut.
5. **Akurasi Interpretasi Gambar (Visual Reasoning):**
   - Diagram geometri sederhana: target pemahaman $\ge 80\%$.
   - Grafik kurva linear/kuadrat: target pemahaman $\ge 75\%$.

### 3. Kebijakan Fallback ke Jalur A (OCR-First)
- Jika perangkat tidak memenuhi syarat vision atau pengguna memilih tidak mengunduh model vision:
  - Aplikasi tetap memproses teks soal menggunakan **Jalur A (OCR-First)**.
  - Untuk bagian soal yang memuat diagram visual yang tidak terbaca, antarmuka menampilkan badge peringatan transparan: *"Soal ini mengandung diagram visual. Fitur teks model mungkin membutuhkan Anda menuliskan nilai sudut/panjang yang tertera pada diagram secara manual."*
  - **Dilarang keras** mengirimkan gambar ke cloud sebagai fallback.

## Konsekuensi
### Positif
- Menjaga integritas teknis: tidak menjanjikan kemampuan visual tanpa pengujian empiris nyata.
- Jalur masa depan yang jelas untuk penanganan soal diagram STEM tanpa mengorbankan stabilitas MVP.
- Zero budget shock dan privasi 100% terjaga.

### Negatif / Mitigasi
- Pengguna dengan soal bergambar murni di perangkat low-end harus memasukkan keterangan gambar secara manual (dimitigasi dengan UI OCR review editor yang mudah diedit).
