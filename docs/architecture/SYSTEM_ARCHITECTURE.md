# Arsitektur Sistem (System Architecture) — KerjaiAja
## Hybrid Offline-First Productivity Platform: Supabase Cloud Services + Optional On-Device Google Gemma AI

Dokumen ini menjelaskan rancangan arsitektur sistem tingkat tinggi aplikasi **KerjaiAja** pasca-rebaseline arsitektur hybrid.

---

## 1. Prinsip Desain Sistem (System Design Principles)

1. **Dual Experience Architecture:**
   - **Mode A (Produktivitas Penuh Tanpa AI):** Seluruh fitur produktivitas personal, deterministik prioritas (0–100), kalender, sinkronisasi cloud Supabase (jika login), dan kolaborasi tim berfungsi 100% tanpa model AI.
   - **Mode B (Asisten Kerja On-Device Gemma Opsional):** Model Google Gemma (1B / 270M) berjalan langsung di perangkat via LiteRT-LM untuk analisis tugas kontekstual (*non-generic chatbot*).
2. **Offline-First & Local-Write-First:**
   - Setiap operasi CRUD ditulis langsung ke Room Database lokal terlebih dahulu (respons seketika milidetik).
   - Sinkronisasi cloud ke Supabase PostgreSQL berjalan di background melalui WorkManager `SyncWorker` dengan antrean `sync_pending_queue`.
3. **Supabase Managed BaaS (Direct Client via `supabase-kt`):**
   - Menggunakan SDK resmi `io.github.jan-tennert.supabase:bom:3.9.0` (Ktor Android Engine).
   - Tidak ada custom backend (Go/Gin, Node.js, atau Python server ditiadakan).
   - Otorisasi dan isolasi data ditegakkan di level database via PostgreSQL Row Level Security (RLS).
4. **Zero AI Server & Zero External AI API:**
   - Tidak ada panggilan ke Gemini API, OpenAI API, Anthropic, atau server AI eksternal apa pun.
   - Privasi 100% terjamin: catatan akademik tidak pernah meninggalkan ponsel untuk keperluan AI.
5. **Device Capability Management:**
   - Evaluasi hardware multi-faktor sebelum mengizinkan pengunduhan model (RAM bebas $\ge 1.5\text{ GB}$, 64-bit ABI, storage $\ge 3\text{ GB}$, thermal aman, baterai $\ge 30\%$).
6. **Cost Guardrails Rp0 (Supabase Free Tier):**
   - Desain database dan sinkronisasi menjamin biaya operasional Rp0 ($0) dalam batas Free Tier. Model AI dilarang disimpan di Supabase Storage.

---

## 2. Diagram Arsitektur Sistem Tingkat Tinggi

```mermaid
flowchart TD
    subgraph DeviceZone ["Perangkat Fisik Android (Client Device)"]
        subgraph PresentationLayer ["Lapisan Presentasi (UI / MVVM)"]
            ComposeUI["Jetpack Compose UI\n(Neo-Bauhaus M3 / Adaptive)"]
            ViewModels["Android ViewModels\n(StateFlow / Unidirectional Data Flow)"]
            ComposeUI <--> ViewModels
        end

        subgraph DomainLayer ["Lapisan Logika Bisnis (Domain Layer)"]
            TaskUseCase["Task & Breakdown Use Cases"]
            PriorityEngine["Deterministic Priority Engine\n(Formula Skor 0-100 & Urgensi)"]
            TaskRouter["Task Classification Router\n(Lightweight Rule-Based Router)"]
            DeviceManager["Device Capability Manager\n(Per-Model & Hardware Check)"]
            AssistantUseCase["Asisten Kerja Use Cases\n(5 Contextual Scenarios)"]
            SyncUseCase["Sync & Conflict Resolution Use Case"]
            
            ViewModels --> TaskUseCase
            ViewModels --> PriorityEngine
            ViewModels --> TaskRouter
            ViewModels --> DeviceManager
            ViewModels --> AssistantUseCase
            ViewModels --> SyncUseCase
        end

        subgraph OnDeviceAIZone ["Lapisan On-Device AI Opsional (LiteRT-LM & OCR)"]
            OcrEngine["ML Kit OCR v2 Engine\n(Text Recognition from Camera/Gallery)"]
            RagPipeline["Local RAG Retriever\n(SQLite FTS5 BM25 + Token Budget <= 800)"]
            LifecycleManager["Model Lifecycle Manager\n(Lazy Load, Single Mutex, Auto-Unload 5m)"]
            LiteRTEngine["LiteRT-LM Engine\n(Google Gemma 3 1B / 270M)"]
            SubjectValidator["Subject Adaptive Validator\n(7-Part Schema, Arithmetic & SI Units)"]
            StructuredParser["Structured Output Parser\n(kotlinx.serialization)"]
            
            TaskRouter -- "Perlu OCR" --> OcrEngine
            TaskRouter -- "Perlu AI" --> RagPipeline
            RagPipeline --> LifecycleManager
            LifecycleManager --> LiteRTEngine
            LiteRTEngine --> SubjectValidator
            SubjectValidator --> StructuredParser
            StructuredParser --> AssistantUseCase
        end

        subgraph StorageLayer ["Lapisan Penyimpanan Lokal (Room Database)"]
            RoomDB[("Room Database 2.6.1\n(Tasks, Subjects, Teams, Schedules)")]
            FTS5Index[("SQLite FTS5 Virtual Table\n(tasks_fts - BM25 Search)")]
            SyncQueue[("sync_pending_queue\n(Pending Mutations Queue)")]
            PrivateStorage[("Private Sandbox Storage\n(context.noBackupFilesDir/models/)")]
            
            TaskUseCase <--> RoomDB
            RagPipeline <--> FTS5Index
            RoomDB -. "SQLite Trigger" .-> FTS5Index
            SyncUseCase <--> SyncQueue
            LiteRTEngine <--> PrivateStorage
        end

        subgraph BackgroundSync ["WorkManager Background Engine"]
            SyncWorker["SyncWorker (WorkManager)\n(Exponential Backoff & Connectivity Check)"]
            SyncQueue <--> SyncWorker
        end
    end

    subgraph WebAdminZone ["Web Admin CMS Environment (Next.js 15 Vercel)"]
        AdminBrowser["Browser Administrator\n(Desktop / Tablet)"]
        EdgeMiddleware["Next.js Edge Middleware\n(Session Refresh & Route Guard)"]
        AdminServer["Server Components & Actions\n(@supabase/ssr + Zod Validation)"]
        
        AdminBrowser <-->|HTTPS Cookie Session| EdgeMiddleware
        EdgeMiddleware --> AdminServer
    end

    subgraph SupabaseCloud ["Supabase Managed BaaS (Free Tier)"]
        SupaAuth["Supabase Auth\n(Guest Migration, Email/Magic Link)"]
        SupaPostgres[("PostgreSQL Database\n(Row Level Security / RLS)")]
        SupaRealtime["Supabase Realtime\n(WebSockets CDC / Team Collaboration)"]
        SupaStorage["Supabase Storage\n(Task Attachments <= 5MB)"]
        
        SyncWorker <-->|PostgREST HTTPS| SupaPostgres
        ViewModels <-->|Auth PKCE| SupaAuth
        ViewModels <-->|Realtime WS| SupaRealtime
        TaskUseCase <-->|Storage API| SupaStorage
        AdminServer <-->|Secure SSR Cookie Client\n(admin_roles RLS)| SupaPostgres
        AdminServer <-->|Auth Admin Session| SupaAuth
    end

    subgraph ExternalCDN ["Distribusi Resmi Model AI"]
        GoogleCDN["Google AI Edge / Hugging Face CDN\n(Official Quantized Gemma Weights)"]
        DeviceManager -. "On-Demand Download (Wi-Fi)" .-> GoogleCDN
        GoogleCDN --> PrivateStorage
    end
```

---

## 3. Komponen & Subsistem Utama

### 3.1 Lapisan Presentasi (Presentation Layer)
- **Framework UI:** Jetpack Compose dengan arsitektur **Neo-Bauhaus** (Material 3).
- **Adaptif Mode A & Mode B:** UI secara dinamis menampilkan kontrol AI jika perangkat memenuhi kualifikasi dan model terpasang, atau menyajikan antarmuka produktivitas bersih (Mode A) tanpa error.
- **State Management:** MVVM + UDF dengan `StateFlow` dan penanganan status error/loading yang elegan.

### 3.2 Lapisan Domain (Domain Layer)
- `DeterministicPriorityEngine`: Menghitung skor prioritas dasar (0–100) matematis transparan.
- `DeviceCapabilityManager`: Menilai status hardware (RAM, ABI, storage, thermal, baterai) menjadi tier: `RECOMMENDED`, `LIMITED`, `NOT_RECOMMENDED`, `UNSUPPORTED`.
- `AsistenKerjaUseCase`: Menjalankan 5 skenario kerja terstruktur: *Ask About My Work*, *Task Breakdown*, *Smart Planning*, *Contextual Recommendations*, *Work Review*.
- `SyncManager`: Mengelola siklus antrean mutasi offline dan resolusi konflik.

### 3.4 Lapisan On-Device AI, OCR & Pemecah Soal (Photo Question Solver)
- **Runtime Inferensi:** Google AI Edge **LiteRT-LM** (Kotlin API) untuk model Google Gemma 3 1B IT (Quantized INT4) atau Gemma 270M.
- **Engine OCR On-Device:** Google ML Kit Text Recognition v2 (`com.google.mlkit:text-recognition:16.0.1`) untuk ekstraksi teks instan dari citra.
- **Klasifikasi Tugas Cerdas (`TaskClassificationRouter`):** Router aturan deterministik yang mengalihkan $>90\%$ aksi langsung ke mesin deterministik tanpa memanggil model LLM.
- **Siklus Hidup Model (`ModelLifecycleManager`):** Mengatur lazy initialization, single active session via `Mutex`, batas timeout 15 detik, timer auto-unload 5 menit, dan respon pembersihan memori LMK.
- **Layar Human-in-the-Loop Review:** Editor teks dengan toolbar simbol akademik matematika/sains dan pemilih 12 rumpun mata pelajaran.
- **Validasi Adaptif (`SubjectAdaptiveValidator`):** Memvalidasi skema solusi 7 bagian, integritas persamaan aritmetika, dan konsistensi satuan SI untuk rumpun Eksakta.
- **Jalur B (Laboratorium Multimodal):** Evaluasi eksperimental model vision multimodal **Gemma 3n E2B** khusus untuk perangkat berkualifikasi tinggi (`VisionCapability = EXPERIMENTAL_QUALIFIED`).
- **Local RAG:** Pencarian leksikal cepat BM25 via tabel virtual `tasks_fts` di Room dengan batas token ketat $\le 800$ token.

### 3.5 Lapisan Data Lokal & Cloud (Hybrid Storage)
- **Room Database 2.6.1:** Basis data lokal sebagai sumber kebenaran instan (*Local-First*).
- **`sync_pending_queue`:** Tabel antrean mutasi untuk mencatat perubahan yang belum dikirim ke cloud.
- **Ephemeral Image Storage:** Berkas citra sementara foto soal disimpan di `context.cacheDir/temp_solver/` dan langsung dihapus seketika setelah ekstraksi/inferensi (zero image in database, zero image to cloud).
- **Supabase PostgreSQL & PostgREST:** Basis data terpusat dengan penegakan RLS.
- **Supabase Realtime:** Sinkronisasi instan untuk kolaborasi tim.
- **Supabase Storage:** Penyimpanan lampiran file dokumen tugas akademik pengguna ($\le 5\text{ MB}$).

### 3.6 Lapisan Web Admin CMS (Next.js 15 App Router & SSR)
- **Framework & Runtime:** Next.js 15, React 19, TypeScript strict mode, Tailwind CSS & shadcn/ui.
- **Autentikasi & RLS Server:** Menggunakan pustaka resmi `@supabase/ssr` berbasis cookie HttpOnly aman. Route guarding di Edge Middleware (`/admin/*`).
- **Pusat Operasional Terpadu:** Manajemen silabus kurikulum 12 mata pelajaran (`academic_references`), respon tiket keluhan dan bug report (`user_feedbacks`), pemantauan kesehatan kuota Free Tier, dan audit trail otomatis (`admin_audit_logs`).
- **Anti-Surveillance & Privasi:** Administrator dibatasi secara ketat oleh RLS sehingga dilarang membaca isi tugas atau percakapan privat mahasiswa.

---

## 4. Keamanan & Biaya Operasional

1. **Zero Exposure of Admin Keys:** Aplikasi Android hanya memegang `anon_key` publik; `service_role_key` dilarang keras disertakan baik di aplikasi Android maupun client bundle Web CMS.
2. **PostgreSQL RLS:** Isolasi data multi-tenant ditegakkan di level database.
3. **Biaya Operasional Rp0:** Pemanfaatan Supabase Free Tier secara disiplin dengan guardrails ketat (zero AI cloud compute, delta sync, auto-cleanup) dan Vercel Hobby untuk hosting Web Admin non-komersial.

---

## 5. Referensi Putusan Arsitektur (Architectural Decision Records)

Arsitektur KerjaiAja ditopang oleh 41 ADR resmi yang tercatat di [Master ADR Index](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/ARCHITECTURE_DECISIONS.md), dengan fokus arsitektur AI, Web CMS, dan Knowledge Intelligence terbaru:
- [ADR-033: Progressive AI Processing Pipeline](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-033-PROGRESSIVE-AI-PROCESSING-PIPELINE.md)
- [ADR-034: On-Device Model Lifecycle & Memory Management](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-034-ON-DEVICE-MODEL-LIFECYCLE-AND-MEMORY-MANAGEMENT.md)
- [ADR-035: Per-Model Device Capability Evaluation](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-035-PER-MODEL-DEVICE-CAPABILITY-EVALUATION.md)
- [ADR-036: Web Admin CMS Architecture and Technology Stack](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-036-WEB-ADMIN-CMS-ARCHITECTURE.md)
- [ADR-037: CMS Role-Based Access Control & Server-Side Security](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-037-CMS-RBAC-AND-SERVER-SIDE-SECURITY.md)
- [ADR-038: Shared Database Schema & Multi-Tenant RLS for CMS](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-038-SHARED-DATABASE-SCHEMA-AND-CMS-RLS.md)
- [ADR-039: Two-Tier Knowledge Intelligence Layer Architecture](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-039-TWO-TIER-KNOWLEDGE-INTELLIGENCE-LAYER.md)
- [ADR-040: Document Ingestion, Intelligent Chunking & Traceability](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-040-DOCUMENT-INGESTION-INTELLIGENT-CHUNKING-TRACEABILITY.md)
- [ADR-041: Context Engineering, Prompt Injection Defense & Knowledge Sync](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-041-CONTEXT-ENGINEERING-PROMPT-INJECTION-AND-KNOWLEDGE-SYNC.md)



