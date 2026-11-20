# Standar Kode & Konvensi Rekayasa (Coding Standards) — KerjaiAja

Dokumen ini mendefinisikan standar penulisan kode, konvensi penamaan, dan aturan rekayasa untuk **KerjaiAja** pasca-rebaseline arsitektur penuh (Full Architecture Rebaseline 2026-11-20), mencakup kode **Kotlin / Jetpack Compose** dan **Go**.

---

## 1. Konvensi Kotlin & Jetpack Compose (Android Client)

### 1.1 Gaya Penulisan & Penamaan
1. Mengikuti panduan resmi *Android Kotlin Style Guide*.
2. Mematuhi analisis statis `ktlint` dan `detekt`.
3. Penamaan:
   - **Classes & Interfaces:** `PascalCase` (misal: `TaskRepository`, `CalculateTaskPriorityUseCase`).
   - **Functions & Methods:** `camelCase` (misal: `calculateScore()`, `observeTasks()`).
   - **Properties & Variables:** `camelCase` (misal: `dueTimestamp`, `selectedSubjectId`).
   - **Constants & Enums:** `UPPER_SNAKE_CASE` (misal: `DEFAULT_PAGE_SIZE`, `TaskPriority.CRITICAL`).
   - **Compose Components:** `PascalCase` kata benda dengan prefiks Neo-Bauhaus jika relevan (misal: `BauhausTaskCard`, `BauhausPriorityPill`).

### 1.2 Praktik Terbaik Jetpack Compose
1. **State Hoisting & Unidirectional Data Flow (UDF):**  
   Komponen UI menerima immutable UI State dan memancarkan user intent/event ke ViewModel via lambda callback.
2. **Modifier Discipline:**  
   Setiap fungsi `@Composable` publik wajib menyediakan parameter `modifier: Modifier = Modifier` di urutan pertama parameter opsional.
3. **Immutability:**  
   UI State dimodelkan dengan `data class` yang seluruh propertinya bertipe *read-only* (`val`) atau koleksi `ImmutableList` untuk memaksimalkan *recomposition skipping*.
4. **Neo-Bauhaus Design Tokens:**  
   Dilarang keras melakukan hardcoding nilai warna heksadesimal atau ukuran dp/sp di dalam Composable. Selalu gunakan token tema dari `KerjaiAjaTheme.colors` dan `KerjaiAjaTheme.typography`.

---

## 2. Konvensi Bahasa Go (Backend Service)

### 2.1 Gaya Penulisan & Format
1. Kode Go wajib diformat menggunakan `gofmt` atau `goimports`.
2. Mematuhi linter resmi `golangci-lint` (errcheck, gosimple, govet, ineffassign, staticcheck).
3. Penamaan:
   - **Packages:** Huruf kecil tunggal tanpa underscore (misal: `task`, `team`, `priority`, `config`).
   - **Exported Symbols:** `PascalCase` (misal: `CalculateScore`, `NewServer`, `TaskHandler`).
   - **Internal/Private Symbols:** `camelCase` (misal: `validateInput`, `dbPool`).
   - **Interfaces:** Berorientasi aksi dengan akhiran *er* jika tepat (misal: `TaskRepository`, `AiProvider`).

### 2.2 Penanganan Error Terstruktur (Error Handling)
1. **Always Check Errors:** Dilarang mengabaikan error (dilarang menulis `_ = db.QueryRow(...)`).
2. **Error Wrapping:** Gunakan `fmt.Errorf("context message: %w", err)` untuk mempertahankan *error chain* asli sehingga dapat diinspeksi via `errors.Is` atau `errors.As`.
3. **Structured Logging:** Gunakan `log/slog` bawaan Go untuk mencatat event dan error dalam format JSON dengan atribut kontekstual (`request_id`, `user_id`, `error`).

---

## 3. Disiplin Git & Manajemen Repositori

1. **Conventional Commits:**  
   Gunakan format commit standar: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`.
2. **No Secret in Git:**  
   Dilarang keras melakukan commit terhadap berkas `.env`, kredensial, token API, atau private key ke repositori Git. Selalu gunakan template `.env.example` untuk variabel lingkungan.
3. **Evidence Before Assertions:**  
   Setiap perubahan kode wajib dibuktikan kelulusannya dengan menjalankan test suite (`go test ./...` atau `./gradlew test`) sebelum menyatakan tugas selesai.
