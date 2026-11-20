# ADR-009: Python Batch-Based Fine-Tuning Pipeline

## Status
ACCEPTED (2026-11-20)

## Konteks
Untuk meningkatkan kualitas penalaran AI KerjaiAja dalam konteks akademik lokal (seperti sistem penugasan perguruan tinggi di Indonesia, terminologi KRS, SKS, praktikum, sidang, dan bahasa percakapan mahasiswa), model dasar (*foundation model*) perlu diadaptasi.

Namun, menjalankan inferensi fine-tuned secara on-device di Android membebani memori dan storage pengguna (memerlukan 2-4 GB model weight). Selain itu, training model real-time atau continuous learning di backend runtime tanpa evaluasi kurasi dapat menyebabkan *catastrophic forgetting* atau kebocoran data pribadi.

## Keputusan
KerjaiAja mengadopsi arsitektur **Python Batch-Based Offline Fine-Tuning Pipeline**:
1. **Pemisahan Pipeline Training dari Runtime Produksi:**
   - Runtime backend Go mengoperasikan model inferensi stabil self-hosted via private daemon `llama.cpp` (seperti Qwen3-1.7B / Qwen3-4B GGUF backend service sesuai [ADR-013](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-013-SELF-HOSTED-AI-LLAMACPP-QWEN.md)).
   - Seluruh aktivitas data preparation, filtering, deduplikasi, augmentasi, LoRA/QLoRA training, dan evaluasi dijalankan secara terpisah menggunakan lingkungan Python (PyTorch / HuggingFace Transformers / PEFT / Unsloth).
2. **Batch Cadence & Dataset Curation:**
   - Training dilakukan secara berkala (batch cycle, misalnya bulanan atau per rilis versi), bukan streaming per event pengguna.
   - Dataset hanya bersumber dari data sintetis akademik terverifikasi dan data pengguna yang memberikan persetujuan eksplisit (*opt-in*).
3. **Standar Evaluasi Sebelum Promosi Model (Safety & Quality Gate):**
   - Sebelum bobot model baru dideploy ke backend produksi, model wajib lolos benchmark evaluasi:
     - Prioritization Accuracy Metric (Spearman Rank Correlation $\ge 0.85$ terhadap anotasi pakar akademik).
     - JSON Schema Compliance Rate ($100\%$ validasi format JSON output).
     - Safety & PII Redaction Check ($0\%$ data identitas mahasiswa yang bocor).
4. **Target Adaptasi Parameter-Efficient:**
   - Menggunakan LoRA (Low-Rank Adaptation) atau QLoRA untuk efisiensi komputasi dan kemudahan penyimpanan adapter checkpoint (<100MB per checkpoint).

## Konsekuensi
### Positif
- Model backend dapat ditingkatkan kemampuannya secara terukur tanpa memengaruhi performa server transaksi live.
- Sumber daya komputasi pelatihan dapat disewa atau dijalankan secara spot/on-demand (misalnya Google Colab / Kaggle / GPU spot instance) tanpa biaya infrastruktur tetap yang mahal.
- Kualitas dan keamanan keluaran model terverifikasi secara deterministik sebelum promosi ke tahap produksi.

### Negatif / Mitigasi
- Terdapat jeda waktu antara umpan balik pengguna dan adaptasi model baru.
  - *Mitigasi:* Menggunakan *few-shot in-context learning* dan prompt engineering adaptif di backend Go untuk kebutuhan personalisasi cepat pengguna secara langsung.
