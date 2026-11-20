# ADR-040: Document Ingestion, Intelligent Chunking, and Source Traceability Engine

## Status
**ACCEPTED & LOCKED** (2026-11-20) — Menetapkan arsitektur penyerapan dokumen, strategi chunking cerdas, dan sitasi terverifikasi.

---

## 1. Konteks (Context)
Model RAG tradisional sering menggunakan *fixed-size chunking* (misal memotong setiap 500 karakter). Pendekatan sembrono ini merusak rumus matematika di tengah formula, memisahkan pertanyaan dari penjelasan pendukung, serta menghilangkan baris header dari tabel angka. Selain itu, model AI rentan menghasilkan sitasi palsu (*hallucinated citations*) jika sistem tidak melacak asal-usul dokumen secara ketat.

---

## 2. Keputusan (Decision)
1. **Intelligent Structure-Aware Chunking:**
   - Menolak *fixed-size chunking*.
   - Menggunakan segmentasi cerdas berorientasi struktur: mempertahankan hierarki judul bab/bagian (`section_title`), menjaga integritas blok rumus matematika LaTeX tanpa pemotongan, dan menyertakan header tabel pada setiap potongan tabel.
   - Ukuran target chunk 150–300 kata dengan overlap 15–20%.
2. **Metadata & Content Hash Tracking:**
   - Setiap dokumen dan chunk dilacak dengan hash SHA-256 (`content_hash`) untuk mencegah duplikasi berkas.
   - Menyimpan nomor halaman, nama bab, dan sumber rujukan resmi.
3. **Penegakan Penelusuran Sumber (Source Traceability):**
   - Jawaban model wajib menyertakan rujukan asli (nama buku/dokumen, bab, halaman, dan kutipan konsep terverifikasi).
   - Melarang sistem mengarang nomor halaman fiktif. Jika data halaman tidak tersedia, tampilkan nama judul bab dan dokumen saja.

---

## 3. Konsekuensi (Consequences)
### Positif:
- Rumus dan tabel tersimpan utuh dan dapat dipahami secara akurat oleh model AI.
- Menghilangkan halusinasi sitasi; mahasiswa dapat memverifikasi jawaban langsung ke buku teks resminya.
- Mencegah duplikasi data materi berkat verifikasi hash SHA-256.
### Negatif:
- Algoritma parsing membutuhkan logika pemeriksaan token dan delimiter yang lebih teliti dibandingkan pemotongan teks sederhana.
