# Strategi Pengujian (Testing Strategy) — KerjaiAja
## Hybrid Offline-First Testing · Deterministic Engine · OCR & Vision Tests · Subject Benchmarks

Dokumen ini mendefinisikan piramida pengujian otomatis, metodologi pengujian, serta pendekatan verifikasi menyeluruh untuk ekosistem **KerjaiAja** pasca-final rebaseline arsitektur hybrid dan integrasi fitur Foto Soal & Pembahasan.

---

## 1. Piramida Pengujian Sistem (Android Native Testing Pyramid)

```text
          / \
         / E2E \         ~10% (End-to-End User Flow, Offline Sync Cycle, Supabase BaaS)
        /-------\
       / Compose \       ~20% (Compose Widget Tests, Neo-Bauhaus UI, OCR Review Screen)
      /   Widget  \
     /-------------\
    /   Unit Tests  \    ~70% (Deterministic Engine, DeviceCapability, OCR Logic, AnswerValidator, Room)
   /-----------------\
```

---

## 2. Rincian Lapisan Pengujian

### 2.1 Unit Tests Android (~70%)
- **Target Pengujian:**
  1. `DeterministicPriorityEngine`: Rumus prioritas (0–100) dan klasifikasi 4 tingkat urgensi (overdue, deadline mendesak, bobot nilai 0–100%).
  2. `DeviceCapabilityManager`: Evaluasi terpisah untuk `TextCapability` (`BASIC_TEXT`, `FULL_TEXT`, `UNSUPPORTED`) dan `VisionCapability` (`EXPERIMENTAL_QUALIFIED`, `NONE`) serta evaluasi per-model (ADR-035).
  3. `TaskClassificationRouter`: Klasifikasi deterministik instan vs perutean ke AI tanpa memanggil LLM classifier.
  4. `ModelLifecycleManager`: Pengujian state transitions (`UNLOADED` -> `LOADING` -> `READY` -> `INFERRING`), proteksi single active session `Mutex`, idle auto-unload 5 menit, dan timeout 15 detik.
  5. `SubjectAdaptiveValidator` & `AnswerValidator`:
     - Verifikasi kelengkapan skema terstruktur 7 bagian.
     - Verifikasi ekspresi aritmetika dasar secara deterministik (`A [op] B = C`).
     - Deteksi inkonsistensi satuan SI (misal: Joule, Newton, m/s).
     - Deteksi ketidakpastian (*uncertaintyFlag*).
     - Ketahanan parser regex fallback jika JSON rusak.
  6. `OcrConfidenceEvaluator`:
     - Evaluasi confidence score per kata dan threshold peringatan ketidakjelasan teks.
  7. `SyncQueue & ConflictResolver`: Enqueue mutasi lokal, resolusi konflik Last-Write-Wins (LWW), versioning, dan soft-deletion tombstone.
  8. `LocalRagRetriever & BM25QueryBuilder`: Query FTS5 BM25 dan pemangkasan token budget ($\le 800$ token).
  9. `ImagePrivacyCleaner`: Memverifikasi bahwa fungsi pembersih cache fisik menghapus file temporer dari disk.
  10. `Room DAOs`: Pengujian kueri DAO in-memory (`Room.inMemoryDatabaseBuilder`).
- **Status Unit Test Baseline:** 41 pengujian unit pada scaffold logika inti telah dieksekusi dan tervalidasi lulus 100% (`41 passed in app/src/test/`).
- **Pustaka Utama:**
  - `JUnit 4` / `JUnit 5` test runner.
  - `MockK`: Mocking repositori dan interface.
  - `Turbine`: Pengujian emisi Kotlin `Flow` reaktif.
  - `kotlinx-coroutines-test`: Pengujian asinkron via `runTest`.

### 2.2 Compose Widget & Interaction Tests (~20%)
- **Target Pengujian:**
  - Rendering komponen Neo-Bauhaus (`BauhausTaskCard`, `BauhausPriorityPill`, `BauhausButton`).
  - Adaptabilitas Dual Experience: Memastikan UI Mode A tetap bersih dan fungsional tanpa tombol AI error jika perangkat `UNSUPPORTED`.
  - **Layar Review OCR (Human-in-the-Loop)**:
    - Input teks dapat diedit di `OutlinedTextField`.
    - Toolbar simbol akademik menyisipkan karakter matematika dengan benar.
    - Selector 12 Rumpun Mata Pelajaran memilih chip yang tepat.
  - Layar Pembahasan Soal: Menampilkan 7 bagian solusi dengan benar dan rendering formula LaTeX/teks.
- **Pustaka Utama:** `createComposeRule()` dari `androidx.compose.ui.test`.

### 2.3 Integration, Sync & BaaS Tests (~10%)
- **Target Pengujian:**
  - **Offline-First Sync Cycle:** Memverifikasi alur mutasi lokal -> `sync_pending_queue` -> mock PostgREST client -> update status sync.
  - **Zero Image Upload Verification:** Memastikan selama proses foto soal berlangsung, tidak ada panggilan jaringan apa pun ke endpoint Supabase Storage.
  - **Guest to Cloud Migration:** Memverifikasi transaksi atomic Room saat mengubah `userId` dari guest ke UID Supabase.
  - **FTS5 Triggers:** Memverifikasi sinkronisasi otomatis antara tabel `tasks` dan `tasks_fts` saat insert, update, dan soft-delete.
  - **Supabase Realtime Simulation:** Memverifikasi konsumsi event WebSocket CDC untuk tugas kelompok tim.

---

## 3. Suite Pengujian Khusus Fitur Foto Soal & Pembahasan

### A. Answer Validator Test Suite
```kotlin
@Test
fun `validator fails when final answer is blank`() {
    val invalidSolution = StructuredSolution(
        questionIdentification = "Hitung kecepatan",
        givenInformation = listOf("s = 100 m", "t = 10 s"),
        questionTarget = "Kecepatan v",
        conceptsAndFormulas = listOf("v = s / t"),
        stepByStepSolution = listOf("v = 100 / 10 = 10 m/s"),
        finalAnswer = "", // KOSONG -> INVALID
        verificationAndCheck = "Pemeriksaan dimensi konsisten"
    )
    assertFalse(invalidSolution.isValid())
}

@Test
fun `arithmetic verifier detects calculation discrepancy`() {
    val step = "Maka hasil kali massa dan percepatan adalah F = 5 * 4 = 25 N"
    val result = AnswerValidator.verifyStepArithmetic(step)
    assertFalse(result.isConsistent)
    assertEquals("5 * 4 = 20", result.suggestedCorrection)
}
```

### B. Subject Benchmark Quality Suite
- Pengujian otomatis berkala terhadap dataset 240 soal pada 12 rumpun mata pelajaran (`docs/quality/SUBJECT_BENCHMARK.md`).
- Menghasilkan laporan metrik: Akurasi Konsep, Validitas Rumus, Kelengkapan 7 Bagian, dan Kualitas Bahasa.
