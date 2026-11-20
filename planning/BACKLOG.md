# Daftar Backlog Produk (Product Backlog) — KerjaiAja
## Hybrid Offline-First Platform · Supabase Managed BaaS · Optional On-Device Gemma AI

Dokumen ini memetakan seluruh item *backlog* pengembangan fungsional dan teknis aplikasi **KerjaiAja** pasca-rebaseline arsitektur hybrid.

---

## 1. Modul Core: Manajemen Tugas & Mata Kuliah (TASK)

- [ ] **TASK-01:** Setup basis data lokal Room 2.6.1 (`TaskEntity`, `SubjectEntity`, `SubtaskEntity`) dan DAO reaktif berbasis Kotlin `Flow`.
- [ ] **TASK-02:** Implementasi tabel virtual SQLite FTS5 (`tasks_fts`) dengan triggers sinkronisasi otomatis untuk pencarian leksikal cepat.
- [ ] **TASK-03:** Repository pattern lokal (`TaskRepository`, `SubjectRepository`) dengan prinsip Local-Write-First.
- [ ] **TASK-04:** UI: Layar Dasbor Tugas dengan filter status, pengelompokan mata kuliah, dan indikator beban kerja.
- [ ] **TASK-05:** UI: Layar Tambah & Edit Tugas dengan pemilih tanggal/jam, bobot nilai SKS, dan estimasi durasi menit.

---

## 2. Modul: Supabase Managed BaaS & Autentikasi (SUPA)

- [ ] **SUPA-01:** Integrasi pustaka resmi Supabase Kotlin SDK v3.9.0 (`auth-kt`, `postgrest-kt`, `realtime-kt`, `storage-kt`) via Ktor Android.
- [ ] **SUPA-02:** Implementasi alur **Guest-First Onboarding**: aplikasi dapat langsung digunakan tanpa login.
- [ ] **SUPA-03:** Alur Autentikasi Cloud Supabase (Email/Password & Magic Link) di menu Profil / Akun.
- [ ] **SUPA-04:** Migrasi Akun Tamu ke Cloud: transaksi atomic Room untuk memperbarui `userId` dari `local_guest_...` ke UID Supabase.
- [ ] **SUPA-05:** Setup Storage Bucket `task_attachments` (Private) dengan batas berkas 5 MB dan penegakan RLS.

---

## 3. Modul: Sinkronisasi Offline-First & Resolusi Konflik (SYNC)

- [ ] **SYNC-01:** Implementasi entitas `sync_pending_queue` di Room untuk mencatat seluruh operasi mutasi lokal (`INSERT`, `UPDATE`, `DELETE`).
- [ ] **SYNC-02:** Implementasi `SyncWorker` (WorkManager) dengan batasan jaringan terhubung dan exponential backoff retry.
- [ ] **SYNC-03:** Logika Resolusi Konflik: Last-Write-Wins (LWW) dengan UTC monotonic timestamp dan version counter.
- [ ] **SYNC-04:** Mekanisme Soft-Deletion: implementasi kolom `deleted_at` (tombstone) agar entitas yang dihapus saat offline disinkronkan secara konsisten.
- [ ] **SYNC-05:** Delta Sync: Klien hanya meminta perubahan remote dengan filter `updated_at > :lastSyncTime` untuk menghemat kuota egress Free Tier.

---

## 4. Modul: Deterministic Priority Engine & Anti-Guilt UX (PRIO)

- [ ] **PRIO-01:** Implementasi `DeterministicPriorityEngine`: formula matematis 4 faktor (0–100) dan 4 kategori urgensi (*Critical, High, Medium, Low*).
- [ ] **PRIO-02:** Tampilan transparansi (*explainable rationale*) yang merinci faktor waktu sisa, bobot nilai, dan kesulitan.
- [ ] **PRIO-03:** Fitur *User Priority Override* di antarmuka Android untuk kendali manual pengguna.
- [ ] **PRIO-04:** Desain & implementasi Anti-Guilt UX untuk penanganan tugas terlambat (*overdue*): opsi adaptif pemulihan jadwal.

---

## 5. Modul: Device Capability Management (DEV)

- [ ] **DEV-01:** Implementasi `DeviceCapabilityManager`: evaluasi multi-faktor RAM bebas ($\ge 1.5\text{ GB}$), arsitektur 64-bit ABI, storage internal ($\ge 3.0\text{ GB}$), thermal status, dan level baterai ($\ge 30\%$).
- [ ] **DEV-02:** Klasifikasi 4 Tingkatan Perangkat: `RECOMMENDED`, `LIMITED`, `NOT_RECOMMENDED`, `UNSUPPORTED`.
- [ ] **DEV-03:** UI: Dialog transparansi spesifikasi perangkat sebelum mengunduh model AI.
- [ ] **DEV-04:** Runtime Monitor: listener `PowerManager.getThermalStatus()` dan `onTrimMemory` untuk auto-unload model.

---

## 6. Modul: On-Device Gemma & Asisten Kerja (ASST)

- [ ] **ASST-01:** Integrasi Google AI Edge **LiteRT-LM** melalui antarmuka `AiEngine` (Gemma 3 1B IT & Gemma 270M).
- [ ] **ASST-02:** Implementasi `LocalRagRetriever`: ekstraksi konteks tugas relevan dari `tasks_fts` menggunakan BM25 dan pruning token budget ($\le 800$ token).
- [ ] **ASST-03:** Implementasi 5 use-case Asisten Kerja:
  - *Ask About My Work* (Tanya beban kerja tugas)
  - *Task Breakdown* (Dekomposisi tugas besar menjadi sub-tugas)
  - *Smart Planning* (Saran alokasi blok waktu belajar)
  - *Contextual Recommendations* (Saran tindakan prioritas berikutnya)
  - *Work Review* (Pemeriksaan checklist pra-pengumpulan)
- [ ] **ASST-04:** `StructuredOutputValidator`: parsing respons model via `kotlinx.serialization` dengan regex auto-repair dan fallback deterministik.
- [ ] **ASST-05:** `ModelDownloadManager`: unduhan model on-demand dari CDN resmi Google AI Edge / Hugging Face dengan verifikasi SHA-256 dan hak hapus instan.

---

## 7. Modul: Kolaborasi Tim & Realtime (TEAM)

- [ ] **TEAM-01:** Skema database PostgreSQL & Room untuk tabel tim (`teams`, `team_members`, `team_tasks`).
- [ ] **TEAM-02:** Penegakan PostgreSQL Row Level Security (RLS) berbasis `auth.uid()` dan relasi `team_members`.
- [ ] **TEAM-03:** Integrasi Supabase Realtime WebSocket (`realtime-kt`) untuk broadcast pembaruan status tugas kelompok.
- [ ] **TEAM-04:** Visualisasi pembagian beban kerja anggota tim (*workload distribution*).

---

## 8. Modul: Sistem Desain Neo-Bauhaus (DES)

- [ ] **DES-01:** Konfigurasi tema Material 3 Neo-Bauhaus di `core/designsystem` (warna fungsional, tipografi Inter, border geometris tegas).
- [ ] **DES-02:** Komponen `BauhausTaskCard`, `BauhausPriorityPill`, `BauhausButton`.
- [ ] **DES-03:** Lembar Bawah Asisten Kerja (*Asisten Kerja BottomSheet*) untuk verifikasi dan persetujuan pengguna.

---

## 9. Modul: Kepatuhan Play Store, Keamanan & Biaya Rp0 (SEC)

- [ ] **SEC-01:** Konfigurasi Target SDK 35 (`compileSdk = 35`, `targetSdk = 35`, `minSdk = 26`) dan R8 release shrinking.
- [ ] **SEC-02:** Fitur Penghapusan Akun Lengkap (*Account Deletion*) sesuai regulasi Google Play Store dengan cascading delete di PostgreSQL.
- [ ] **SEC-03:** Verifikasi Kepatuhan Kuota Supabase Free Tier: Database $\le 500\text{ MB}$, Storage $\le 1\text{ GB}$, Egress $\le 2\text{ GB/bln}$.
- [ ] **SEC-04:** Audit Keamanan Android Keystore (AES-256-GCM) untuk token sesi Supabase Auth.

---

## 10. Modul: Foto Soal & Pembahasan (SOLV)

- [ ] **SOLV-01:** Integrasi CameraX dan Android Photo Picker dengan batas memori piksel ($\le 1920\times 1080$) dan proteksi OOM.
- [ ] **SOLV-02:** Integrasi Google ML Kit Text Recognition v2 (`com.google.mlkit:text-recognition:16.0.1`) dengan confidence score per kata.
- [ ] **SOLV-03:** UI: Layar Peninjauan Teks (*Human-in-the-Loop Review*) dengan toolbar simbol akademik (matematika, fisika, kimia, Yunani).
- [ ] **SOLV-04:** UI: Selector 12 Rumpun Mata Pelajaran untuk injeksi konteks *prompt* akademik.
- [ ] **SOLV-05:** Implementasi `AnswerValidator`: validasi skema 7 bagian, verifikasi aritmetika deterministik, konsistensi satuan, dan uncertainty flagging.
- [ ] **SOLV-06:** UI: Layar Solusi Pembahasan Terstruktur 7 Bagian dengan kartu Neo-Bauhaus dan rendering notasi akademik.
- [ ] **SOLV-07:** Mekanisme pembersihan efemeral berkas citra dari `context.cacheDir` (zero cloud image upload).
- [ ] **SOLV-08:** Pipeline evaluasi eksperimental model multimodal **Gemma 3n E2B** untuk Jalur B Lab khusus perangkat berkualifikasi tinggi (`VisionCapability = EXPERIMENTAL_QUALIFIED`).

---

## 11. Modul: Progressive AI Pipeline & Lifecycle (PIPE)

- [ ] **PIPE-01:** Implementasi `TaskClassificationRouter`: klasifikasi deterministik instan vs perutean ke AI tanpa memanggil LLM classifier.
- [ ] **PIPE-02:** Implementasi `ModelLifecycleManager`: state machine transisi status (`UNLOADED`, `LOADING`, `READY`, `INFERRING`, `THROTTLED`), proteksi single active session `Mutex`, idle auto-unload 5 menit, dan timeout 15 detik.
- [ ] **PIPE-03:** Implementasi `SubjectAdaptiveValidator`: validasi struktur 7 bagian adaptif per 12 mata pelajaran, verifikasi aritmetika, dan konsistensi satuan SI.
- [ ] **PIPE-04:** Implementasi evaluasi kapabilitas per-model di `DeviceCapabilityManager` (ADR-035).
- [ ] **PIPE-05:** Benchmark performa terarah pada perangkat Low-End, Mid-Range, dan High-End (`AI_PERFORMANCE_BENCHMARK.md`).

---

## 12. Modul: Web Admin CMS (CMS)

- [ ] **CMS-01:** Setup arsitektur Next.js 15 App Router, TypeScript strict mode, Tailwind CSS, dan shadcn/ui.
- [ ] **CMS-02:** Integrasi Supabase SSR Auth (`@supabase/ssr`) dengan route guarding ketat di Edge Middleware (`/admin/*`).
- [ ] **CMS-03:** UI: Dashboard Ringkasan Operasional (Metrik pengguna, tim aktif, tiket feedback, status kuota Supabase Free Tier).
- [ ] **CMS-04:** UI & Logika: Manajemen Status Akun Pengguna (Pencarian ID, filter status, penangguhan akun, tanpa intip konten privat).
- [ ] **CMS-05:** UI & Logika: Manajemen Master Kurikulum 12 Rumpun Mata Pelajaran & Silabus (`academic_references`, `curriculum_topics`).
- [ ] **CMS-06:** UI & Logika: Pengelolaan Tiket Umpan Balik & Evaluasi Laporan OCR (`user_feedbacks`).
- [ ] **CMS-07:** Penegakan Jejak Audit Administratif otomatis pada tabel `admin_audit_logs`.
- [ ] **CMS-08:** Pengujian Otomatis CMS (Validasi skema Zod, isolasi RLS database, dan Playwright E2E smoke tests).

---

## 13. Modul: Knowledge Intelligence Layer (KNOW)

- [ ] **KNOW-01:** Pemisahan Dua Domain Pengetahuan: tabel virtual `tasks_fts` (Productivity) & `academic_fts` (Academic) di Room Database.
- [ ] **KNOW-02:** Document Ingestion Pipeline: validasi berkas, ekstraksi teks (MD, TXT, OCR), normalisasi, dan hash SHA-256 anti-duplikasi.
- [ ] **KNOW-03:** Intelligent Structure-Aware Chunking: pemotongan cerdas dengan preservasi rumus matematika LaTeX dan header tabel.
- [ ] **KNOW-04:** Query Understanding Engine: normalisasi input, klasifikasi domain, ekstraksi kata kunci, dan penyusunan sintaks FTS5 BM25.
- [ ] **KNOW-05:** Context Engineering: pembangun konteks khusus (Productivity max 800 tokens, Academic max 1200 tokens) dan pertahanan Prompt Injection.
- [ ] **KNOW-06:** Source Traceability: pelacakan sitasi terverifikasi (nama buku, bab, nomor halaman, kutipan) dan pencegahan sitasi halusinasi.
- [ ] **KNOW-07:** Offline-First Delta Knowledge Sync: sinkronisasi delta referensi kurikulum dari CMS ke Android dengan deletion markers dan kontrol penyimpanan pengguna (<100 MB).
- [ ] **KNOW-08:** RAG Evaluation & Golden Test Suite: pengujian metrik Retrieval (P@K, R@K, MRR) dan metrik Kualitas Jawaban (Groundedness, Faithfulness, Abstention).



