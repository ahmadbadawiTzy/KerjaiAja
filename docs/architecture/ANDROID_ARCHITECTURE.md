# Arsitektur Aplikasi Android (Android Architecture) — KerjaiAja
## Clean Architecture · MVVM · UDF · Supabase BaaS · Optional On-Device Gemma

Dokumen ini menjelaskan struktur arsitektur teknis aplikasi **Android Native** untuk **KerjaiAja** pasca-rebaseline arsitektur hybrid.

---

## 1. Prinsip Dasar Arsitektur

1. **Clean Architecture Separation of Concerns:**
   - **Presentation Layer (UI):** Menampilkan antarmuka pengguna berbasis Jetpack Compose dengan prinsip Neo-Bauhaus, mengamati `StateFlow`, dan mendukung Dual Experience (Mode A & Mode B).
   - **Domain Layer:** Menampung aturan bisnis murni (*UseCases*), formula prioritas deterministik, evaluasi kapabilitas perangkat, dan algoritma resolusi konflik data. Bebas dari dependensi Android framework.
   - **Data Layer:** Mengelola persistensi data lokal (*Room Database 2.6.1 & FTS5*), antrean mutasi offline (*sync_pending_queue*), dan remote client Supabase (*supabase-kt* v3.9.0).
2. **Offline-First & Local-Write-First:**
   - Database Room lokal adalah sumber kebenaran instan (*Immediate Source of Truth*). Operasi CRUD ditulis ke Room dan antrean sinkronisasi terlebih dahulu.
   - Sinkronisasi cloud ke Supabase PostgreSQL berjalan di background melalui `SyncWorker` (WorkManager).
3. **Unidirectional Data Flow (UDF):**
   - State mengalir ke bawah (*downward*) dari ViewModel ke Composable functions.
   - User action/events mengalir ke atas (*upward*) dari Composable ke ViewModel.
4. **Zero Custom Backend & Zero External AI API:**
   - Aplikasi berkomunikasi langsung dengan Supabase Managed BaaS via PostgREST, Auth, Realtime, dan Storage.
   - AI dijalankan 100% on-device menggunakan Google Gemma via LiteRT-LM (opsional).

---

## 2. Struktur Modul & Paket Aplikasi

```text
com.kerjaiaja.app/
├── core/
│   ├── common/               # Result wrapper, AppDispatchers, Extensions, Constants
│   ├── database/             # Room Database, Type Converters, Migrations, FTS5 Setup
│   ├── datastore/            # UserPreferences, DeviceTierState DataStore
│   ├── designsystem/         # Neo-Bauhaus Theme, Tokens, Shapes, Icons, Components
│   ├── device/               # DeviceCapabilityManager, DeviceTier, HardwareMetrics
│   ├── sync/                 # SyncQueue, SyncStatus, ConflictResolver, SyncWorker
│   ├── solver/               # QuestionSubject, StructuredSolution, AnswerValidator, OcrEngine
│   └── security/             # Android Keystore Manager, Session Token Encryption
│
├── ai/
│   ├── engine/               # AiEngine (Interface), LiteRtLmEngine, FakeAiEngine
│   ├── rag/                  # LocalRagRetriever, BM25QueryBuilder, TokenBudgetPruner
│   ├── prompt/               # GemmaPromptTemplates, StrictJsonSchemaRules
│   ├── validator/            # StructuredOutputValidator, AnswerValidator
│   └── manager/              # ModelDownloadManager, Sha256Verifier, ModelLifecycleManager
│
├── data/
│   ├── local/
│   │   ├── dao/              # TaskDao, SubjectDao, TeamDao, SyncQueueDao, FtsSearchDao
│   │   └── entity/           # Room Entity Models (TaskEntity, SubjectEntity, SyncQueueEntity)
│   ├── remote/
│   │   ├── client/           # SupabaseClientProvider (Auth, Postgrest, Realtime, Storage)
│   │   ├── dto/              # Supabase DTO Data Classes (@Serializable)
│   │   └── datasource/       # SupabaseTaskDataSource, SupabaseTeamDataSource, SupabaseAuthDataSource
│   └── repository/           # Repository Implementations (OfflineFirstTaskRepository, TeamRepository, AuthRepository)
│
├── domain/
│   ├── model/                # Pure Kotlin Domain Entities (Task, Subject, Team, DeviceTier, AsistenResponse)
│   ├── repository/           # Repository Interfaces (Contracts)
│   └── usecase/
│       ├── task/             # GetTasksUseCase, CreateTaskUseCase, UpdateTaskStatusUseCase
│       ├── priority/         # DeterministicPriorityEngine, OverridePriorityUseCase
│       ├── device/           # EvaluateDeviceCapabilityUseCase
│       ├── assistant/        # AskAboutMyWorkUseCase, TaskBreakdownUseCase, SmartPlanningUseCase
│       ├── solver/           # ProcessQuestionImageUseCase, ValidateSolutionUseCase
│       ├── sync/             # EnqueueMutationUseCase, TriggerSyncUseCase, ResolveConflictUseCase
│       └── team/             # GetTeamTasksUseCase, SubscribeTeamRealtimeUseCase
│
└── presentation/
    ├── navigation/           # Type-Safe Navigation Compose Graph & Destinations
    ├── common/               # Shared Neo-Bauhaus UI Components (Buttons, Cards, Badges)
    ├── dashboard/            # Academic Dashboard & Priority Matrix Screen & ViewModel
    ├── tasks/                # Personal Task CRUD & Detail Screen & ViewModel
    ├── assistant/            # Asisten Kerja Bottom Sheet & Action ViewModels
    ├── solver/               # CameraX Capture, OCR Review, 7-Section Solution Screen & ViewModel
    ├── subjects/             # Subject & Class Schedule Screen & ViewModel
    ├── team/                 # Team Workspace & Realtime Collaboration Screen & ViewModel
    ├── focus/                # Focus Timer (Pomodoro) Screen & ViewModel
    └── settings/             # Settings, Supabase Account, Model Download Manager Screen & ViewModel
```

---

## 3. Rincian Lapisan Arsitektur

### 3.1 Presentation Layer (UI & MVVM)
- **Jetpack Compose:** Desain **Neo-Bauhaus** (Material 3) dengan geometri fungsional dan kontras tinggi.
- **Dual Experience Handling:** ViewModel memeriksa `DeviceCapability` dan `ModelStatus`. Jika mode AI belum aktif atau perangkat `UNSUPPORTED`, UI menyajikan Mode A (Produktivitas Penuh) tanpa memunculkan error atau placeholder rusak.
- **StateFlow & UDF:** Seluruh state UI berupa immutable data class yang dikonsumsi via `collectAsStateWithLifecycle()`.

### 3.2 Domain Layer (Core Business Logic)
- **Deterministic Priority Engine:** Perhitungan skor matematis transparan (0–100) berbasis urgensi waktu dan bobot nilai.
- **Device Capability Manager:** Evaluasi RAM bebas, arsitektur 64-bit, storage internal, thermal status, dan level baterai.
- **Asisten Kerja Use Cases:** Menangani 5 use case kontekstual terstruktur tanpa chatbot generic.
- **Conflict Resolver:** Logika resolusi konflik Last-Write-Wins (LWW) dan version checking saat sinkronisasi cloud.

### 3.3 Data Layer (Hybrid Storage)
- **Room Database (Local-First):** Menyimpan data lokal secara instan dan mencatat mutasi ke `sync_pending_queue`.
- **Supabase Remote Integration:**
  - `Postgrest`: Melakukan sync perubahan lokal ke PostgreSQL via REST.
  - `Realtime`: Menerima update real-time untuk tugas kelompok tim via WebSocket CDC.
  - `Storage`: Mengunggah lampiran dokumen akademik pengguna ($\le 5\text{ MB}$).
  - `Auth`: Menangani migrasi akun tamu ke akun cloud Supabase.

---

## 4. Konkurensi & Dependency Injection

- **Kotlin Coroutines & Flow:**
  - `Dispatchers.Main`: Khusus UI Compose.
  - `Dispatchers.IO`: Database Room, akses file model disk, dan Supabase PostgREST network calls.
  - `Dispatchers.Default`: Inferensi LiteRT-LM, rumus matematika prioritas, dan JSON serialization.
- **WorkManager:**
  - `SyncWorker`: Menjalankan sinkronisasi background saat terhubung internet dengan exponential backoff retry.
- **Hilt Dependency Injection:**
  - `@Singleton`: `RoomDatabase`, `SupabaseClient`, `DeviceCapabilityManager`, `DeterministicPriorityEngine`.
  - `@ViewModelScoped`: Dependencies terikat lifecycle ViewModel.
