# Desain Kontrak Antarmuka Domain & Integrasi Supabase Client — KerjaiAja
## Internal Domain Contracts · Supabase BaaS Client · On-Device AI Schemas

- **Status Dokumen:** BASELINE LOCKED
- **Arsitektur:** Hybrid Offline-First (Supabase Managed BaaS + Optional On-Device Gemma)
- **Cakupan:** Kontrak Antarmuka Domain Kotlin, Operasi Supabase PostgREST & Realtime, serta Skema JSON Asisten Kerja

---

## 1. Transisi Arsitektur API

Aplikasi KerjaiAja **tidak menggunakan custom backend server** (tidak ada Go/Gin/Node/Python REST API). Komunikasi sistem terbagi menjadi dua level:
1. **Internal Domain Contracts:** Antarmuka Kotlin murni untuk orkestrasi logika bisnis di dalam aplikasi Android.
2. **Supabase Client Operations:** Panggilan PostgREST, Realtime WebSocket, Storage, dan Auth menggunakan pustaka resmi `io.github.jan-tennert.supabase:bom:3.9.0`.

---

## 2. Kontrak Antarmuka Repositori Domain (Domain Contracts)

### 2.1 `TaskRepository` Interface
```kotlin
interface TaskRepository {
    fun getTasksStream(filter: TaskFilter): Flow<List<Task>>
    suspend fun getTaskById(taskId: String): Task?
    suspend fun upsertTask(task: Task): Result<Unit>
    suspend fun deleteTask(taskId: String): Result<Unit>
    suspend fun getSubtasksForTask(taskId: String): List<Subtask>
    suspend fun upsertSubtasks(subtasks: List<Subtask>): Result<Unit>
    suspend fun updateTaskStatus(taskId: String, status: TaskStatus): Result<Unit>
    suspend fun overrideTaskPriority(taskId: String, priorityLevel: PriorityLevel): Result<Unit>
}
```

### 2.2 `SyncManager` & `SyncQueue` Interface
```kotlin
interface SyncManager {
    val syncStatus: StateFlow<SyncStatus>
    suspend fun enqueueMutation(entityType: String, entityId: String, operation: SyncOperation, payloadJson: String)
    suspend fun triggerSyncNow(): Result<Unit>
    suspend fun processPendingQueue(): Result<Int>
}
```

### 2.3 `DeviceCapabilityManager` Interface
```kotlin
interface DeviceCapabilityManager {
    fun evaluateCapability(): DeviceCapability
    fun getDeviceTier(): DeviceTier
    fun isModelDownloadSupported(): Boolean
}
```

### 2.4 `DeterministicPriorityEngine` Interface
```kotlin
interface DeterministicPriorityEngine {
    fun calculateScore(
        deadlineMillis: Long,
        academicWeightPercent: Double,
        difficulty: Int,
        estimatedMinutes: Int,
        hasUnfinishedDependencies: Boolean
    ): PriorityResult // Mengembalikan skor (0-100) dan PriorityLevel
}
```

---

## 3. Spesifikasi Operasi Supabase Client (PostgREST & Realtime)

### 3.1 Operasi PostgREST Incremental Sync
Klien Android menyinkronkan data tugas menggunakan query delta dengan parameter `updated_at`:
```kotlin
// Tarik perubahan remote setelah timestamp sinkronisasi terakhir
val remoteTasks = supabase.from("tasks")
    .select {
        filter {
            gt("updated_at", lastSyncTimestampUtc)
        }
    }
    .decodeList<TaskDto>()

// Unggah mutasi lokal dari sync_pending_queue
supabase.from("tasks").upsert(taskDto)
```

### 3.2 Langganan Realtime untuk Kolaborasi Tim
```kotlin
val channel = supabase.channel("team_channel_$teamId")
val changeFlow = channel.postgresChangeFlow<PostgresAction>(schema = "public") {
    table = "team_tasks"
    filter = "team_id=eq.$teamId"
}

channel.subscribe()
changeFlow.collect { action ->
    when (action) {
        is PostgresAction.Insert -> handleRemoteTaskInserted(action.decodeRecord<TeamTaskDto>())
        is PostgresAction.Update -> handleRemoteTaskUpdated(action.decodeRecord<TeamTaskDto>())
        is PostgresAction.Delete -> handleRemoteTaskDeleted(action.oldRecord)
        else -> Unit
    }
}
```

---

## 4. Skema JSON Kontrak Asisten Kerja On-Device

Setiap output inferensi on-device Google Gemma divalidasi ketat menggunakan `kotlinx.serialization`.

### 4.1 Skema Dekonstruksi Tugas (`TaskBreakdownResponse`)
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "TaskBreakdownResponse",
  "type": "object",
  "properties": {
    "subtasks": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "title": { "type": "string" },
          "estimated_minutes": { "type": "integer", "minimum": 10, "maximum": 180 },
          "sort_order": { "type": "integer" }
        },
        "required": ["title", "estimated_minutes", "sort_order"]
      }
    },
    "reasoning_summary": { "type": "string" }
  },
  "required": ["subtasks", "reasoning_summary"]
}
```

### 4.2 Skema Tanya Pekerjaan (`AskAboutWorkResponse`)
```json
{
  "summary": "Terdapat 2 tugas kritis yang jatuh tempo dalam 24 jam ke depan.",
  "critical_task_ids": ["uuid-1", "uuid-2"],
  "recommended_action": "Kerjakan Laporan Praktikum Fisika terlebih dahulu karena memiliki bobot 20% dan estimasi durasi 90 menit."
}
```
