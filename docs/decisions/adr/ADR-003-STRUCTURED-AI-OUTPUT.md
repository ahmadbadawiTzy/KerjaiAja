# ADR-003: Strategi Keluaran Terstruktur AI dan Pertahanan Berlapis (Structured AI Output Strategy & Defense-in-Depth)

> [!NOTE]
> **Status:** **CONSOLIDATED INTO ADR-027 & ADR-031** (Rancangan Awal Pra-Rebaseline)  
> Dokumen ini merupakan rancangan awal strategi parsing output terstruktur AI dan validasi JSON. Seluruh keputusan dan implementasinya telah dilebur dan diformalkan ke dalam:
> - **[ADR-027: Photo Question Solver Specification](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-027-PHOTO-QUESTION-SOLVER-SPECIFICATION.md)**
> - **[ADR-031: Academic Answer Validation and Quality Control](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-031-ACADEMIC-ANSWER-VALIDATION-AND-QUALITY-CONTROL.md)**  
> Nomor kanonikal ADR-003 pada master index dialokasikan untuk **[ADR-003: Supabase Managed Infrastructure](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-003-SUPABASE-MANAGED-INFRASTRUCTURE.md)** (Status: SUPERSEDED oleh ADR-020).

- **Status:** CONSOLIDATED INTO ADR-027 & ADR-031
- **Tanggal:** 2026-11-20
- **Pengambil Keputusan:** Tim Arsitektur KerjaiAja (Disetujui Pengguna)
- **Terkait Dokumen:** [`docs/decisions/adr/ADR-027-PHOTO-QUESTION-SOLVER-SPECIFICATION.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-027-PHOTO-QUESTION-SOLVER-SPECIFICATION.md), [`docs/decisions/adr/ADR-031-ACADEMIC-ANSWER-VALIDATION-AND-QUALITY-CONTROL.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-031-ACADEMIC-ANSWER-VALIDATION-AND-QUALITY-CONTROL.md), [`docs/ai/AI_EVALUATION_STRATEGY.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/ai/AI_EVALUATION_STRATEGY.md)

---

## 1. Konteks (Context)

Keluaran asisten AI pada KerjaiAja digunakan secara langsung untuk mengisi daftar periksa sub-tugas (*subtask checklist*) dan rubrik verifikasi pengumpulan (*submission checklist*). Oleh karena itu, respon teks mentah dari model bahasa harus selalu dapat dikonversi menjadi objek data terstruktur Kotlin (`List<SubtaskDomainModel>`).

Audit dokumentasi (`docs/audits/SUPERPOWERS_DOCUMENTATION_AUDIT.md` temuan AUD-HIGH-02) menunjukkan bahwa mengandalkan klaim target kepatuhan skema JSON `>= 98%` semata-mata dari rekayasa prompt teks pada model kecil (Gemma 3 1B INT4) tidak realistis. Model kompak berbobot kuantisasi 4-bit rentan menyisipkan karakter rusak, teks obrolan pembuka/penutup, atau pembungkus markdown (` ```json `).

## 2. Masalah (Problem)

Jika aplikasi langsung mewajibkan fitur *constrained grammar decoding* tanpa memverifikasi ketersediaan dan kestabilan API tersebut pada SDK resmi LiteRT-LM, arsitektur menjadi rapuh (*fragile dependency*). Sebaliknya, jika aplikasi hanya mengandalkan serializer standar tanpa pertahanan parsing, satu karakter tidak valid akan memicu *unhandled exception* atau kegagalan fitur di mata pengguna.

## 3. Keputusan (Decision)

Mengadopsi **Verifikasi Runtime Bertahap dan Pertahanan Berlapis (*Defense-in-Depth Structured Output*)**:

1. **Verifikasi Kemampuan Runtime LiteRT-LM:**
   - Memeriksa secara berkala kemampuan resmi pustaka `com.google.ai.edge.litertlm:litertlm-android` pada rilis yang ditargetkan.
   - Mengelompokkan status fitur runtime:
     - *Tersedia & Stabil:* Langsung diadopsi jika API JSON schema/grammar constraint telah stabil di Android.
     - *Eksperimental:* Diisolasi di balik flag konfigurasi uji coba (*feature toggle*).
     - *Belum Didukung:* Mengandalkan pipeline pembersihan defensif di tingkat aplikasi.
2. **Pipeline Pertahanan Berlapis (*Defense-in-Depth Pipeline*):**
   - **Layer 1 (Strict System Prompt):** Template prompt instruksi sistem yang melarang teks basa-basi dan mewajibkan format JSON polos.
   - **Layer 2 (Sanitization & Fault-Tolerant Pre-Parser):** Lapisan pra-parsing berbasis ekspresi reguler yang membersihkan blok markdown ticks, memotong karakter pengantar sebelum tanda kurung siku/kurawal pertama (`[` atau `{`), serta menghapus *trailing comma*.
   - **Layer 3 (Strict Schema Validation):** Parsing menggunakan `kotlinx.serialization` dengan konfigurasi `ignoreUnknownKeys = true`.
   - **Layer 4 (Bounded Retry):** Jika parsing tetap gagal pada percobaan pertama, sistem melakukan maksimal **1 kali percobaan ulang (*bounded retry*)** dengan prompt penegasan format.
   - **Layer 5 (Deterministic Fallback):** Jika seluruh percobaan gagal, sistem tidak boleh crash atau menampilkan pesan teknis menyeramkan; aplikasi secara transparan menampilkan antarmuka pembuatan sub-tugas manual konvensional (*fail-safe manual mode*).
3. **Pelabelan Target Metrik yang Jujur:**
   - Menghapus klaim angka validitas 98% tanpa bukti empiris. Seluruh target akurasi skema diberi label **`REQUIRES TECHNICAL VALIDATION`** hingga dataset uji nyata (50 ragam tugas akademik) dievaluasi di perangkat fisik.

## 4. Alternatif yang Dipertimbangkan (Alternatives Considered)

- **Alternatif A (Mewajibkan Constrained Decoding Sejak Awal):** Ditolak karena API grammar decoding pada edge runtime mobile masih terus berkembang pesat dan berpotensi memicu masalah kompatibilitas antar chipset.
- **Alternatif B (Menelan Error & Menampilkan Layar Kosong):** Ditolak karena melanggar prinsip keandalan produk (*reliability*).

## 5. Konsekuensi (Consequences)

### Positif:
- Sistem kebal terhadap variasi keluaran model AI yang tidak terduga.
- Alur kerja pengguna tidak pernah terhenti atau terblokir akibat kegagalan sintaks AI.
- Arsitektur tetap fleksibel untuk mengadopsi native constrained decoding saat API tersebut telah matang.

### Negatif:
- Memerlukan implementasi lapisan *sanitizer* tambahan di data layer (`JsonOutputSanitizer`).

## 6. Risiko & Mitigasi (Risks & Mitigations)

- **Risiko:** Model menghasilkan teks halusinasi yang valid JSON tetapi isinya tidak masuk akal.
- **Mitigasi:** Pengguna selalu diberikan kendali tinjauan akhir (*review & edit*) sebelum rekomendasi sub-tugas disimpan ke database.

## 7. Persyaratan Validasi (Validation Requirements)

- `REQUIRES TECHNICAL VALIDATION`: Pengujian unit test dengan 50 fixture teks output model malformed (markdown ticks, incomplete brackets, conversational preambles) untuk memastikan sanitization pipeline 100% lolos ke fallback atau berhasil diperbaiki.
