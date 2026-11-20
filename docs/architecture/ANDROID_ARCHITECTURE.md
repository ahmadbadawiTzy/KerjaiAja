# Arsitektur Aplikasi Android (Android Architecture) — KerjaiAja

Dokumen ini menjelaskan struktur arsitektur teknis aplikasi **Android Native** untuk **KerjaiAja**, mengacu pada prinsip **Clean Architecture** dan pola **MVVM (Model-View-ViewModel)**.

---

## 1. Prinsip Dasar Arsitektur

1. **Clean Architecture Separation of Concerns:**
   - **UI Layer (Presentation):** Menampilkan data dan menangkap input pengguna.
   - **Domain Layer:** Menyimpan logika bisnis inti aplikasi (bebas dari dependensi platform Android).
   - **Data Layer:** Mengelola pengambilan data dari sumber lokal (Room, DataStore) dan sumber eksternal (AI Model, Supabase).
2. **Aturan Ketergantungan (Dependency Rule):**
   - Ketergantungan kode hanya mengarah ke dalam (*inward dependency*): UI bergantung pada Domain; Data mengimplementasikan kontrak Domain; Domain tidak bergantung pada siapa pun.
3. **Unidirectional Data Flow (UDF):**
   - State mengalir ke bawah (*downward*), event mengalir ke atas (*upward*).

---

## 2. Struktur Modul & Paket Rekomendasi

```text
com.kerjaiaja.app/
├── core/
│   ├── common/               # Utility umum, Result wrapper, Dispatcher providers
│   ├── database/             # Room Database, Type Converters, Migrations
│   ├── designsystem/         # Material 3 Theme, Typography, Reusable Components
│   └── network/              # Supabase Client & HTTP interceptors (optional)
│
├── data/
│   ├── ai/                   # LiteRT-LM runtime wrapper, ModelDownloader, PromptBuilder
│   ├── repository/           # Implementasi Repository (Task, Subject, Group, AI)
│   └── local/
│       ├── dao/              # Room DAOs (TaskDao, SubjectDao, FtsDao)
│       └── entity/           # Room Entities & FTS virtual tables
│
├── domain/
│   ├── model/                # Pure Kotlin domain models
│   ├── repository/           # Repository interfaces (kontrak data)
│   ├── ai/                   # AiEngine interface & AI response models
│   └── usecase/              # Interactors / UseCases per fitur
│
└── presentation/
    ├── navigation/           # Compose Navigation graph & Destinations
    ├── tasks/                # Task list, detail, create/edit screens & ViewModels
    ├── subjects/             # Subject list & detail screens & ViewModels
    ├── breakdown/            # Assignment breakdown interactive screen & ViewModel
    ├── studyplan/            # Study planner & focus timer screen & ViewModel
    ├── group/                # Group project delegation screen & ViewModel
    ├── checklist/            # Submission pre-flight checklist screen & ViewModel
    └── assistant/            # Local AI learning assistant chat & ViewModel
```

---

## 3. Rincian Lapisan Arsitektur

### 3.1 Presentation Layer (UI & MVVM)
- **Framework:** Jetpack Compose deklaratif.
- **State Modeling:** Setiap layar memiliki satu *immutable data class* representasi UI State (misal: `TaskDetailUiState`).
- **ViewModel Lifecycle:** Memanfaatkan Android `ViewModel` yang dikelola oleh Hilt (`@HiltViewModel`).
- **State Exposure:** State diekspos melalui `StateFlow` dan dikonsumsi di Compose menggunakan `collectAsStateWithLifecycle()`.
- **One-off Events:** Notifikasi singkat (*snackbars*, navigasi) ditangani via `Channel` atau `SharedFlow` berorientasi event.

### 3.2 Domain Layer (Core Business Logic)
- **UseCases:** Setiap UseCase hanya memiliki satu tanggung jawab publik (*Single Responsibility*) menggunakan fungsi `operator fun invoke(...)`.
  - Contoh: `BreakdownTaskWithAiUseCase`, `GetUpcomingDeadlinesUseCase`, `ToggleSubmissionChecklistItemUseCase`.
- **Domain Models:** Model data murni yang bebas dari anotasi Room `@Entity` atau serialization JSON. Model ini merefleksikan entitas akademik sesungguhnya.

### 3.3 Data Layer (Persistensi & AI Runtime)
- **Repository Pattern:** Bertindak sebagai mediator yang menyatukan data lokal, hasil inferensi AI, dan status sinkronisasi.
- **Room Database:** Menangani caching lokal, query relasional, dan integrasi SQLite FTS untuk pencarian teks penuh.
- **Local AI Engine Wrapper:** Komponen internal data layer yang memuat berkas bobot model, mengelola antrean inferensi, dan memetakan output teks model menjadi objek domain terstruktur.

---

## 4. Konkurensi & Background Processing

- **Kotlin Coroutines:** Digunakan untuk seluruh operasi asinkron non-pemblokir (*non-blocking*).
- **Dispatchers:**
  - `Dispatchers.Main`: Khusus pembaruan UI dan interaksi Compose.
  - `Dispatchers.IO`: Khusus operasi baca/tulis disk Room dan jaringan.
  - `Dispatchers.Default`: Khusus komputasi intensif dan preprocessing teks prompt.
  - Dispatcher khusus alokasi memori inferensi untuk mencegah starving thread pool standar.
- **WorkManager:** Digunakan untuk eksekusi tugas latar belakang yang persisten, seperti pengunduhan model on-demand yang dapat dilanjutkan (*resumable download*) dan pengingat deadline akademik terjadwal.

---

## 5. Dependency Injection dengan Hilt

- `@InstallIn(SingletonComponent::class)`: Untuk database Room, DataStore, klien jaringan, dan manajer model AI.
- `@InstallIn(ViewModelComponent::class)`: Untuk dependensi yang terikat pada siklus hidup ViewModel spesifik.
- Penyediaan dependensi selalu mengutamakan antarmuka (*interface binding* via `@Binds`), memfasilitasi penggantian dependensi dengan *fake* atau *mock* pada pengujian otomatis.
