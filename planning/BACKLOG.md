# Daftar Backlog Produk (Product Backlog) — KerjaiAja

Dokumen ini memetakan seluruh item *backlog* pengembangan fungsional dan teknis aplikasi **KerjaiAja**, dikelompokkan berdasarkan modul kerja.

---

## 1. Modul Core: Manajemen Tugas (Task Management)

- [ ] **TASK-01:** Desain entitas Room `TaskEntity` dan konverter tipe (`TaskPriority`, `TaskStatus`, `TaskType`).
- [ ] **TASK-02:** Implementasi `TaskDao` (insert, update, delete, getById, observeTasksBySubject, observeUpcomingTasks).
- [ ] **TASK-03:** Implementasi `TaskRepository` dan mappers dari Entity ke Domain model.
- [ ] **TASK-04:** UseCase: `GetUpcomingTasksUseCase`, `CreateTaskUseCase`, `UpdateTaskStatusUseCase`, `DeleteTaskUseCase`.
- [ ] **TASK-05:** UI: Layar daftar tugas utama (*Task Dashboard*) dengan filter status dan prioritas berbasis Jetpack Compose.
- [ ] **TASK-06:** UI: Layar tambah & edit tugas dengan pemilih tanggal (*Date/Time Picker*) dan dropdown mata kuliah.

---

## 2. Modul Core: Manajemen Mata Kuliah (Subject Management)

- [ ] **SUBJ-01:** Desain entitas Room `SubjectEntity` dan `SubjectDao`.
- [ ] **SUBJ-02:** Implementasi `SubjectRepository` dan UseCase (`GetAllSubjectsUseCase`, `CreateSubjectUseCase`).
- [ ] **SUBJ-03:** UI: Layar daftar mata kuliah dengan penanda warna (*color tag*) dan jadwal pertemuan.
- [ ] **SUBJ-04:** UI: Layar detail mata kuliah menampilkan agregasi tugas aktif yang berelasi.

---

## 3. Modul: Dekonstruksi Tugas (Assignment Breakdown)

- [ ] **BRK-01:** Desain entitas Room `SubtaskEntity` dan relasi 1-to-many dengan `TaskEntity`.
- [ ] **BRK-02:** Implementasi `SubtaskDao` dan `SubtaskRepository`.
- [ ] **BRK-03:** UseCase: `AddSubtaskUseCase`, `ToggleSubtaskStatusUseCase`, `ReorderSubtasksUseCase`.
- [ ] **BRK-04:** UI: Komponen *Interactive Stepper Checklist* untuk memvisualisasikan sub-tugas dan persentase progres.
- [ ] **BRK-05:** Mekanisme pembaruan status tugas induk otomatis ketika seluruh sub-tugas tuntas.

---

## 4. Modul: Perencanaan Belajar & Pengatur Fokus (Study Planner)

- [ ] **PLN-01:** Perhitungan alokasi waktu pengerjaan tugas berdasarkan estimasi jam dan batas deadline.
- [ ] **PLN-02:** UI: Layar *Daily Study Agenda* memetakan tugas prioritas hari ini.
- [ ] **PLN-03:** UI & Logika: *Focus Timer* (Pomodoro-friendly) yang terikat langsung ke ID tugas yang sedang aktif.
- [ ] **PLN-04:** Integrasi notifikasi lokal terjadwal untuk pengingat sesi belajar.

---

## 5. Modul: Adaptive Local AI Engine & RAG

- [ ] **AI-01:** Abstraksi antarmuka `AiEngine` dan model data respon AI di domain layer.
- [ ] **AI-02:** Implementasi `DeviceCapabilityDetector` (pemeriksaan RAM bebas, arsitektur CPU, API level).
- [ ] **AI-03:** Integrasi `WorkManager` untuk download berkas model biner on-demand dengan penanganan jeda/lanjut.
- [ ] **AI-04:** Komponen `Sha256Validator` untuk memvalidasi integritas berkas bobot model yang diunduh.
- [ ] **AI-05:** Integrasi tabel virtual SQLite FTS (`academic_notes_fts`) untuk pencarian cepat konteks materi.
- [ ] **AI-06:** Implementasi runtime wrapper `LiteRtLmEngine` (inisialisasi model Gemma 3 1B IT, tokenisasi, streaming output).
- [ ] **AI-07:** Prompt Builder: Template terstruktur untuk dekonstruksi instruksi tugas ke skema JSON.
- [ ] **AI-08:** UI: Alur dialog bantuan AI di layar breakdown tugas dengan tombol *Generate Breakdown* dan *Apply to Checklist*.
- [ ] **AI-09:** Fallback Mechanism: Menonaktifkan fitur AI secara elegan pada perangkat yang tidak memenuhi syarat.

---

## 6. Modul: Manajemen Tugas Kelompok (Group Project)

- [ ] **GRP-01:** Penambahan field delegasi anggota tim pada `SubtaskEntity`.
- [ ] **GRP-02:** UI: Matriks penugasan anggota kelompok dan pelacakan status kontribusi per individu.
- [ ] **GRP-03:** Fitur ekspor ringkasan progres proyek kelompok ke format teks terstruktur untuk dibagikan ke grup pesan (WhatsApp/Telegram).

---

## 7. Modul: Checklist Pengumpulan (Submission Checklist)

- [ ] **CHK-01:** Desain entitas Room `SubmissionChecklistEntity` dan DAO.
- [ ] **CHK-02:** Template verifikasi pra-pengumpulan standar (Format PDF/ZIP, konvensi nama berkas, daftar pustaka).
- [ ] **CHK-03:** UI: Lembar verifikasi *Pre-Flight Submission* sebelum menandai tugas sebagai *Done*.

---

## 8. Modul: Pengaturan & Sinkronisasi Opsional (Settings & Sync)

- [ ] **SET-01:** Pengelolaan tema tampilan (Material You Dynamic Color, Dark Mode, Light Mode) via DataStore.
- [ ] **SET-02:** Antarmuka manajemen penyimpanan berkas model AI (lihat ukuran, hapus berkas model, download ulang).
- [ ] **SET-03:** Inisialisasi klien SDK Supabase untuk otentikasi opsional dan pencadangan data ke cloud PostgreSQL.
