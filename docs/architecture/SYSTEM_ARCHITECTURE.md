# Arsitektur Sistem (System Architecture) — KerjaiAja

Dokumen ini menjelaskan rancangan arsitektur sistem tingkat tinggi aplikasi **KerjaiAja**.

---

## 1. Prinsip Desain Sistem (System Design Principles)

1. **Local-First & Offline-First:**  
   Perangkat pengguna (*Android client*) bertindak sebagai *single source of truth*. Semua data disimpan dan diproses secara lokal di basis data SQLite melalui Room.
2. **Decoupled AI Engine:**  
   Subsistem inferensi kecerdasan buatan lokal diisolasi di balik abstraksi antarmuka (*interface abstraction*). Fitur aplikasi berinteraksi dengan kontrak AI, bukan dengan pustaka runtime spesifik secara langsung.
3. **Graceful Degradation:**  
   Jika perangkat pengguna mengalami keterbatasan memori atau tidak memiliki model AI lokal, sistem secara elegan menonaktifkan fitur AI tanpa mengganggu alur kerja produktivitas manual.
4. **Cloud-Optional Synchronization:**  
   Integrasi cloud (Supabase) bersifat opsional. Tidak ada logika bisnis lokal yang bergantung pada respon jaringan (*zero blocking network dependencies*).

---

## 2. Diagram Blok Arsitektur Tingkat Tinggi

```mermaid
flowchart TD
    subgraph ClientDevice ["Android Device (KerjaiAja Client)"]
        subgraph PresentationLayer ["Presentation Layer"]
            UI["Jetpack Compose UI\n(Material 3, Screens, Components)"]
            VM["Android ViewModels\n(StateFlow, UI States, Events)"]
        end

        subgraph DomainLayer ["Domain Layer (Pure Kotlin)"]
            UC["UseCases / Interactors\n(Business Rules, Validations)"]
            RepoInterfaces["Repository Interfaces"]
            AiInterface["AiEngine Interface"]
        end

        subgraph DataLayer ["Data Layer"]
            RepoImpl["Repository Implementations"]
            
            subgraph LocalDataSources ["Local Data Sources"]
                RoomDB["Room Database\n(Tasks, Subjects, Checklists)"]
                FTS["SQLite FTS4/5\n(Full-Text Search Context)"]
                DataStore["DataStore Preferences\n(User Config & AI State)"]
            end

            subgraph LocalAiSubsystem ["Adaptive Local AI Engine"]
                LiteRt["LiteRT-LM Runtime"]
                ModelManager["Model Manager\n(Download, Verify SHA-256, Cache)"]
                PromptEngine["Prompt Template & Context Injector"]
            end

            subgraph RemoteDataSources ["Remote Data Source (Optional)"]
                SupabaseClient["Supabase SDK Client\n(Auth, PostgREST, Storage)"]
                SyncWorker["WorkManager Sync\n(Background Replication)"]
            end
        end
    end

    subgraph ExternalCloud ["Cloud Services (Optional)"]
        SupabaseBackend[("Supabase Cloud\nPostgreSQL & Storage")]
        ModelCDN["Model Distribution CDN\n(Gemma 3 1B IT Artifacts)"]
    end

    %% Connections
    UI --> VM
    VM --> UC
    UC --> RepoInterfaces
    UC --> AiInterface
    RepoInterfaces -.-> RepoImpl
    AiInterface -.-> LocalAiSubsystem
    
    RepoImpl --> RoomDB
    RepoImpl --> FTS
    RepoImpl --> DataStore
    RepoImpl -.-> RemoteDataSources

    ModelManager --> ModelCDN
    RemoteDataSources -.-> SupabaseBackend
```

---

## 3. Komponen & Subsistem Utama

### 3.1 Presentation Layer
- Menggunakan **Jetpack Compose** deklaratif berbasis komponen **Material 3**.
- **State Hoisting:** Semua state antarmuka dimodelkan secara *immutable* di dalam `ViewModel` menggunakan `StateFlow`.
- Satu arah aliran data (*Unidirectional Data Flow / UDF*): UI mengirimkan *User Intent/Event* ke ViewModel, dan menerima pembaruan *UI State*.

### 3.2 Domain Layer
- Modul Kotlin murni tanpa dependensi pada Android SDK (`android.*`).
- Menampung aturan bisnis esensial dalam bentuk `UseCases` (misal: `BreakdownAssignmentUseCase`, `CalculateStudyScheduleUseCase`, `VerifySubmissionChecklistUseCase`).
- Mendefinisikan antarmuka abstraksi `AiEngine` dan `Repository`.

### 3.3 Data Layer
- **Local Data Source (Room):** Mengelola persistensi relasional lokal dengan enkripsi aman jika diperlukan.
- **SQLite FTS:** Menyediakan kemampuan pencarian teks berkecepatan tinggi pada catatan dan silabus lokal untuk injeksi konteks RAG.
- **Adaptive Local AI Subsystem:** Menangani siklus hidup model AI: manajemen pengunduhan, validasi hash SHA-256, inisialisasi runtime `LiteRT-LM`, dan inferensi token.
- **Optional Remote Data Source:** Menggunakan Supabase client untuk sinkronisasi cadangan data pengguna jika fitur cloud diaktifkan.

---

## 4. Keamanan & Batasan Arsitektural

- **Tidak Ada Hardcoded Secrets:** Token dan konfigurasi disimpan dalam `secrets.properties` lokal yang dikecualikan oleh `.gitignore`.
- **Integritas Model AI:** Bobot model yang diunduh wajib diverifikasi hash SHA-256 sebelum dimuat ke memori untuk mencegah eksploitasi berkas biner rusak atau termodifikasi.
- **Manajemen Memori:** Alokasi memori inferensi dibatasi ketat menggunakan pemanggilan terstruktur (*structured concurrency*) dengan pelepasan sumber daya segera setelah inferensi selesai.
