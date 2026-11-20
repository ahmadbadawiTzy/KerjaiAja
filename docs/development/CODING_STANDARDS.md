# Standar Kode & Konvensi Rekayasa (Coding Standards) — KerjaiAja

Dokumen ini mendefinisikan standar penulisan kode Kotlin, konvensi penamaan, dan aturan rekayasa untuk **KerjaiAja**.

---

## 1. Konvensi Bahasa Kotlin & Format Kode

1. **Gaya Penulisan Resmi:** Mengikuti panduan resmi *Android Kotlin Style Guide*.
2. **Peralatan Linter:** Wajib mematuhi aturan analisis statis `ktlint` dan `detekt`.
3. **Penamaan Berkas & Simbol:**
   - **Classes & Interfaces:** `PascalCase` (misal: `TaskRepository`, `BreakdownAssignmentUseCase`).
   - **Functions & Methods:** `camelCase` (misal: `calculateDueDate()`, `observeTasks()`).
   - **Variables & Properties:** `camelCase` (misal: `dueTimestamp`, `selectedSubjectId`).
   - **Constants & Enums:** `UPPER_SNAKE_CASE` (misal: `MAX_TOKEN_LIMIT`, `TaskPriority.HIGH`).
   - **Compose Components:** `PascalCase` kata benda (misal: `TaskCard`, `SubjectChip`, `SubmissionChecklistRow`).

---

## 2. Praktik Terbaik Jetpack Compose

1. **State Hoisting:**
   - Komponen UI wajib menerima *state* sebagai parameter dan memancarkan *events* via lambda:
     ```kotlin
     @Composable
     fun TaskItemRow(
         task: TaskDomainModel,
         onTaskClick: (String) -> Unit,
         onToggleStatus: (String) -> Unit,
         modifier: Modifier = Modifier
     )
     ```
2. **Modifier Parameter:**
   - Setiap fungsi `@Composable` publik wajib menyediakan parameter `modifier: Modifier = Modifier` di posisi awal parameter opsional.
3. **Immutability:**
   - Seluruh model UI State wajib menggunakan `data class` dengan properti bertipe *read-only* (`val`) atau koleksi `ImmutableList` / `PersistentList` untuk mengoptimalkan *recomposition skipping*.

---

## 3. Disiplin Clean Architecture & Dependency Rules

1. **Domain Layer Terisolasi:**
   - Dilarang mengimpor pustaka Android framework (`android.*`, `androidx.*`) ke dalam modul `domain`, kecuali anotasi yang netral platform.
2. **Penggunaan UseCases:**
   - Satu UseCase = satu fungsi aksi publik (`operator fun invoke(...)`).
   - Hindari membuat UseCase raksasa (*God UseCase*) yang menangani banyak aksi sekaligus.
3. **Dependency Injection:**
   - Semua kelas repository dan service wajib diinjeksikan via konstruktor menggunakan `@Inject constructor(...)`.
   - ViewModel tidak boleh membuat dependensi secara langsung (`new` / manual instantiation).

---

## 4. Konkurensi: Coroutines & Flow

1. **No Hardcoded Dispatchers:**
   - Jangan pernah memanggil `Dispatchers.IO` atau `Dispatchers.Default` secara langsung di dalam UseCase atau Repository. Gunakan abstraksi dispatcher provider yang dapat di-*mock* saat unit testing.
2. **Exception Handling Terstruktur:**
   - Operasi asinkron harus membungkus hasil dalam tipe sealed `Result<T>` atau memanfaatkan `catch` operator pada Kotlin Flow. Dilarang menelan exception tanpa penanganan logis.
3. **Flow Collection di UI:**
   - Selalu konsumsi Flow di layar Compose menggunakan ekstensi `collectAsStateWithLifecycle()`.
