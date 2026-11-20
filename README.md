# KerjaiAja — Academic Productivity Platform

> **Proposisi Nilai Utama:** *"Turn academic assignments into clear, manageable, and organized actions."*  
> Mengubah tugas akademik yang menumpuk dan membingungkan menjadi tindakan yang terstruktur, terarah, dan tuntas.

---

## 1. Ikhtisar Proyek (Project Overview)

**KerjaiAja** adalah aplikasi produktivitas akademik berbasis **Android Native (Kotlin)** yang dirancang khusus untuk membantu pelajar mengelola beban tugas sekolah atau perkuliahan, baik secara mandiri maupun kolaboratif dalam kelompok. Aplikasi ini mengedepankan pendekatan *local-first*, privasi tinggi, performa gegas, serta integrasi kecerdasan buatan lokal (*Adaptive Local AI Engine*) yang dapat berjalan langsung di perangkat (*on-device*) tanpa ketergantungan wajib pada server eksternal.

> [!IMPORTANT]
> **Pernyataan Status Implementasi:**
> Proyek ini saat ini berada pada tahap **Perancangan Arsitektur dan Inisialisasi Workspace Fondasi**. **Implementasi kode aplikasi Android, skrip build Gradle, serta dependensi library belum dimulai.** Seluruh dokumen dalam repositori ini merupakan acuan rekayasa resmi sebelum pengembangan kode dilakukan.

---

## 2. Visi Produk (Product Vision)

Dunia akademik sering kali menuntut pelajar menghadapi beban kognitif tinggi: silabus yang padat, instruksi tugas yang ambigu, tenggat waktu (*deadline*) yang bertumpuk, serta dinamika kerja kelompok yang sering tidak merata.

Visi **KerjaiAja** adalah menjadi asisten kerja akademik yang:
- **Mengurai Kompleksitas:** Mengubah tugas besar yang menakutkan menjadi sub-tugas kecil yang konkret (*actionable micro-steps*).
- **Menjaga Fokus dan Disiplin:** Membantu penyusunan jadwal belajar adaptif dengan pelacakan progres yang transparan.
- **Mendorong Kolaborasi Sehat:** Memfasilitasi pembagian tanggung jawab proyek kelompok tanpa friksi.
- **Menjamin Privasi & Aksesibilitas:** Beroperasi secara *offline-first* dengan kapabilitas AI lokal yang tidak mengorbankan data pribadi pelajar.

---

## 3. Pengguna Sasaran (Target Users) & Karakter Produk

### Pengguna Sasaran
- **Mahasiswa Perguruan Tinggi:** Menghadapi multi-mata kuliah, tugas riset, laporan praktikum, serta tugas kelompok lintas jadwal.
- **Siswa Sekolah Menengah:** Membutuhkan struktur pengerjaan tugas harian dan persiapan ujian terorganisasi.

### Karakter Produk
- **Profesional & Terfokus:** Tampilan bersih, tipografi tegas, bebas dari distraksi elemen gamifikasi yang kekanak-kanakan (*anti-gimmick*).
- **Cepat & Responsif:** Transisi mulus, waktu muat instan, dan konsumsi daya baterai yang efisien.
- **Dapat Diandalkan (Reliable):** Fungsionalitas inti tetap 100% berjalan normal saat koneksi internet mati atau perangkat tidak mendukung akselerasi AI.

---

## 4. Kapabilitas Utama (Core Capabilities)

1. **Academic Task Management:**  
   Pencatatan dan pelacakan tugas akademik dengan atribut lengkap: tenggat waktu (*due date*), skala prioritas, status pengerjaan, label mata kuliah, dan estimasi waktu.
2. **Subject Management:**  
   Pengorganisasian mata kuliah/pelajaran, jadwal tatap muka, dosen/guru pengampu, tautan ruang kelas, serta arsip catatan relevan.
3. **Assignment Breakdown:**  
   Mekanisme dekonstruksi tugas kompleks menjadi hierarki sub-langkah pengerjaan yang realistis dan terukur.
4. **Study Planning:**  
   Penyusunan sesi belajar terjadwal berbasis kalender akademik dan ritme kerja personal.
5. **AI Learning Assistant (Adaptive Local AI):**  
   Asisten penalaran on-device untuk membantu menyusun kerangka penulisan, memecah tugas, serta meringkas materi referensi secara privat tanpa koneksi cloud.
6. **Group Project Management:**  
   Matriks pembagian peran, delegasi sub-tugas kelompok, dan pelacakan kontribusi bersama.
7. **Submission Checklist:**  
   Daftar verifikasi akhir sebelum pengumpulan tugas (format berkas, rubrik penilaian, batas ukuran file, dan kelengkapan lampiran).

---

## 5. Arah Teknologi & Arsitektur (Technology Direction)

Pengembangan aplikasi berpedoman pada standar rekayasa modern Android:

- **Platform:** Android Native
- **Bahasa Utama:** Kotlin (Coroutines, Flow)
- **UI Framework:** Jetpack Compose dengan Material 3 (Material You / Dynamic Color)
- **Pola Arsitektur:** Clean Architecture + MVVM (Model-View-ViewModel)
- **Minimum Versi OS:** Android 10 (API Level 29)
- **Basis Data Lokal:** Room Database dengan SQLite Full-Text Search (FTS)
- **Dependency Injection:** Hilt
- **Background Processing:** WorkManager
- **Backend Cloud (Opsional):** Supabase (untuk otentikasi, sinkronisasi cadangan data, dan kolaborasi daring jika diinginkan pengguna)

---

## 6. Strategi Local AI (Local AI Strategy)

KerjaiAja menerapkan pendekatan **Adaptive Local AI Engine**:

- **Kandidat Model Utama:** **Gemma 3 1B IT** (model instruksi kompak efisien dari Google).
- **Kandidat Runtime Inferensi:** **LiteRT-LM** (Google AI Edge LiteRT for Large Language Models).
- **Alternatif Pendukung:** Gemini Nano melalui Android ML Kit GenAI APIs (pada perangkat yang kompatibel secara sistem).
- **Target Hardware:** Perangkat dengan RAM 4–6 GB (dengan batas alokasi memori inferensi yang terkontrol ketat).
- **Distribusi On-Demand:** Bobot model diunduh secara terpisah setelah instalasi aplikasi (tidak membengkakkan ukuran APK awal) dengan validasi integritas **SHA-256 Checksum**.
- **Local RAG (Retrieval-Augmented Generation):** Menggabungkan konteks catatan dan silabus lokal via Room + SQLite FTS ke dalam prompt sebelum inferensi, memprioritaskan rekayasa prompt di atas fine-tuning model.
- **Zero Mandatory Cloud AI:** Tidak ada kewajiban berlangganan API cloud berbayar atau hosting VPS khusus inferensi. Aplikasi tetap dapat digunakan sepenuhnya tanpa model AI.

---

## 7. Status Pengembangan Saat Ini (Current Status)

| Indikator | Kondisi Terkini |
| :--- | :--- |
| **Tahapan Proyek** | `Fase 0: Inisialisasi Workspace & Perancangan Fondasi` |
| **Source Code Kotlin** | Belum ada (belum dimulai) |
| **Konfigurasi Gradle** | Belum ada (belum di-generate) |
| **Dokumentasi Teknis** | Selesai dan terstruktur penuh di direktori `docs/` dan `planning/` |

---

## 8. Panduan Direktori Dokumentasi (Documentation Guide)

Dokumentasi proyek dikelompokkan secara modular untuk memudahkan navigasi:

```text
KerjaiAja/
├── docs/
│   ├── product/
│   │   ├── PRODUCT_VISION.md           # Visi, proposisi nilai, dan target persona
│   │   ├── PRODUCT_REQUIREMENTS.md     # Spesifikasi kebutuhan fungsional & non-fungsional
│   │   └── MVP_SCOPE.md                # Batasan fitur rilis perdana vs masa depan
│   │
│   ├── architecture/
│   │   ├── SYSTEM_ARCHITECTURE.md      # Arsitektur sistem level tinggi & subsystem layout
│   │   ├── ANDROID_ARCHITECTURE.md     # Penerapan Clean Architecture, MVVM, dan Compose
│   │   └── DATA_ARCHITECTURE.md        # Desain Room DB, SQLite FTS, dan skema Supabase
│   │
│   ├── ai/
│   │   ├── LOCAL_AI_ARCHITECTURE.md    # Sub-sistem Adaptive Local AI Engine & lifecycle
│   │   ├── MODEL_STRATEGY.md           # Pemilihan Gemma 3 1B IT, LiteRT-LM, & guardrails
│   │   ├── MODEL_DISTRIBUTION.md       # On-demand download, SHA-256, & storage management
│   │   └── AI_EVALUATION_STRATEGY.md   # Metrik TTFT, TPS, konsumsi memori, & akurasi
│   │
│   ├── design/
│   │   └── DESIGN_DIRECTION.md         # Desain Material 3, tipografi, & panduan visual
│   │
│   ├── development/
│   │   ├── DEVELOPMENT_ROADMAP.md      # Rencana kerja bertahap dari Fase 0 hingga Rilis
│   │   ├── CODING_STANDARDS.md         # Konvensi penulisan Kotlin, package, dan best practices
│   │   └── TESTING_STRATEGY.md         # Piramida pengujian (Unit, Robolectric, Compose UI)
│   │
│   └── decisions/
│       └── ARCHITECTURE_DECISIONS.md   # Catatan keputusan arsitektural (ADR)
│
├── planning/
│   ├── BACKLOG.md                      # Daftar backlog modul & fitur
│   └── IMPLEMENTATION_PHASES.md        # Rencana aksi eksekusi teknis per fase
│
├── assets/                             # Direktori aset statis (branding, diagram, referensi)
└── research/                           # Catatan riset Android, Local AI, dan analisis kompetitor
```
