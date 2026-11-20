# Strategi Pengujian (Testing Strategy) — KerjaiAja

Dokumen ini mendefinisikan piramida pengujian otomatis, metodologi pengujian, serta pendekatan verifikasi untuk **KerjaiAja**.

---

## 1. Piramida Pengujian (Testing Pyramid)

```text
         / \
        / E2E \         ~10% (End-to-End UI & Room Flow)
       /-------\
      / Compose \       ~20% (Component & Interaction Tests)
     /   Widget  \
    /-------------\
   /   Unit Tests  \    ~70% (UseCases, ViewModels, DAOs, Parsers)
  /-----------------\
```

---

## 2. Tingkatan Pengujian (Testing Levels)

### 2.1 Unit Tests (Prioritas Tertinggi ~70%)
- **Target:** Domain UseCases, ViewModels, Data Mappers, JSON Parsers, dan Room DAOs (menggunakan in-memory database).
- **Pustaka Utama:**
  - `JUnit 5` / `JUnit 4` runner.
  - `MockK`: Untuk mocking dependensi antarmuka.
  - `Turbine`: Untuk menguji emisi Kotlin `Flow` dan `StateFlow` secara deterministik.
  - `kotlinx-coroutines-test`: Untuk memanipulasi waktu virtual (`StandardTestDispatcher`, `runTest`).
- **Prinsip:** Unit test harus berjalan sangat cepat (< 30 detik untuk seluruh suite) tanpa memerlukan emulator atau perangkat fisik Android.

### 2.2 Compose Widget & UI Tests (~20%)
- **Target:** Memvalidasi rendering antarmuka, penanganan state kosong (*empty states*), dan respon interaksi pengguna (klik tombol, input teks, scroll).
- **Pustaka Utama:**
  - `createComposeRule()` dari `androidx.compose.ui.test`.
  - Menguji komponen individual dalam isolasi (*Component-Level Testing*).

### 2.3 End-to-End & Integration Tests (~10%)
- **Target:** Alur kritis pengguna dari ujung ke ujung:
  - Membuat mata kuliah baru ➡️ Menambahkan tugas ➡️ Melakukan breakdown manual ➡️ Menandai checklist selesai.
- **Pustaka Utama:** AndroidX Test Runner pada emulator CI.

---

## 3. Strategi Pengujian Khusus Local AI Engine

Pengujian model kecerdasan buatan lokal memerlukan pendekatan khusus karena bobot biner dan inferensi on-device bersifat *non-deterministik* dan membutuhkan sumber daya komputasi besar:

1. **Abstraksi `FakeAiEngine` untuk CI/CD:**  
   Selama pengujian unit dan otomatisasi CI/CD, implementasi `AiEngine` nyata digantikan oleh `FakeAiEngine` yang mengembalikan respons JSON deterministik tanpa memuat berkas bobot model atau runtime `LiteRT-LM`.
2. **Uji Validitas Parsing JSON:**  
   Dataset fixture berisi 50+ variasi keluaran mentah model LLM (termasuk skenario teks terpotong, format salah, atau markdown formatting) diuji terhadap parser data untuk memastikan aplikasi tidak pernah crash saat menghadapi keluaran tak terduga.
3. **Uji Integritas Hash Checksum:**  
   Pengujian unit khusus untuk memverifikasi bahwa `ModelDownloader` menolak berkas dengan hash SHA-256 yang tidak sesuai dan berhasil menyetujui berkas dengan hash valid.
4. **Benchmarking Manual pada Perangkat Fisik:**  
   Pengujian latensi nyata (TTFT dan TPS) dijalankan terpisah menggunakan instrumentasi benchmark di perangkat fisik nyata.
