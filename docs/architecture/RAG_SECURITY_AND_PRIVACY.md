# RAG Security, Data Isolation & Privacy Architecture
## Penegakan Otorisasi Pra-Retrieval, Isolasi Multi-Tenant, dan Proteksi Anti-Surveillance

- **Status:** APPROVED & LOCKED
- **Lokasi Dokumen:** `docs/architecture/RAG_SECURITY_AND_PRIVACY.md`
- **Terkait:** [`SECURITY_ARCHITECTURE.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/security/SECURITY_ARCHITECTURE.md), [`CMS_SECURITY_AND_RBAC.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/CMS_SECURITY_AND_RBAC.md)

---

## 1. Prinsip Utama Keamanan & Privasi RAG

Arsitektur sistem pengambilan pengetahuan (*Retrieval-Augmented Generation*) di KerjaiAja dibangun di atas fondasi **kedaulatan data dan privasi siswa (UU PDP No. 27/2022)**:
1. **Zero External Data Transmission:** Prompt, dokumen catatan belajar, dan data tugas mahasiswa diproses 100% lokal di perangkat (*On-Device Compute*). Tidak ada transmisi ke API AI eksternal seperti OpenAI, Google Gemini, atau Anthropic.
2. **Pre-Retrieval Authorization:** Validasi hak akses kepemilikan data (*authorization*) wajib dieksekusi di level kueri database (SQL/RLS) **SEBELUM** data ditarik ke dalam memori aplikasi atau diinjeksi ke context builder.
3. **Strict Domain Non-Contamination:** Domain *Productivity Knowledge* dan *Academic Knowledge* diisolasi secara fisik dan logis. Tugas pribadi mahasiswa tidak boleh muncul sebagai referensi soal akademik, dan materi kurikulum tidak boleh mengaburkan prioritas tugas pribadi.
4. **Anti-Surveillance (Anti-Intip Admin):** Administrator CMS tidak memiliki hak akses (*Zero Access*) untuk membaca, menelusuri, atau melihat isi tugas, catatan belajar, atau riwayat foto soal mahasiswa.

---

## 2. Matriks Otorisasi Berdasarkan Lingkup Dokumen (Authorization Matrix)

| Lingkup Dokumen (*Privacy Scope*) | Pemilik / Aktor | Kebijakan Pembacaan Kueri RAG | Lokasi Penyimpanan |
| :--- | :--- | :--- | :--- |
| **`PRIVATE_USER_ONLY`** (Catatan tugas, personal breakdown) | Mahasiswa Individu | `WHERE owner_id = :current_user_id` | Room Database Lokal (Hanya di ponsel pemilik) |
| **`TEAM_SHARED`** (Tugas kelompok, catatan proyek tim) | Anggota Tim Resmi | `WHERE project_id IN (SELECT project_id FROM team_members WHERE user_id = :current_user_id)` | Room Database Lokal + Supabase RLS Team Table |
| **`PUBLIC_STUDENT_BASE`** (Silabus, rumus, referensi 12 mapel) | Sistem / Kurator CMS | `WHERE is_published = 1 AND domain = 'ACADEMIC'` | Room Database Lokal (`academic_chunks`) |
| **`DRAFT_UNPUBLISHED`** (Materi kurikulum yang sedang direview) | Kurator CMS | **DIBLOKIR TOTAL** dari retrieval Android. Hanya terlihat di Web Admin CMS via `is_admin()`. | Supabase PostgreSQL Table (`academic_references`) |

---

## 3. Penegakan Otorisasi Pra-Retrieval (Pre-Retrieval Enforcement)

Sistem **DILARANG KERAS** melakukan kueri luas (*broad query*) lalu menyaring kepemilikan data di lapisan frontend Kotlin atau UI. Kueri Room DAO wajib menyertakan filter kepemilikan dan hak akses secara eksplisit:

```kotlin
@Dao
interface ProductivityRagDao {

    @Query("""
        SELECT t.id, t.title, t.description, t.subject_name, bm25(tasks_fts) AS rank
        FROM tasks t
        JOIN tasks_fts ON t.id = tasks_fts.rowid
        WHERE tasks_fts MATCH :query
          AND t.owner_id = :currentUserId
          AND t.deleted_at IS NULL
          AND (t.is_completed = 0 OR t.completed_at >= :retentionCutoff)
        ORDER BY rank ASC
        LIMIT :limit
    """)
    suspend fun searchAuthorizedPersonalTasks(
        query: String,
        currentUserId: String,
        retentionCutoff: Long,
        limit: Int = 5
    ): List<TaskRagEntity>
}
```

Jika `owner_id != currentUserId`, database Room atau Supabase PostgreSQL secara deterministik mengembalikan 0 baris (*empty result*), mencegah kebocoran data antar-pengguna secara struktural.

---

## 4. Perlindungan Privasi Citra Foto Soal (Ephemeral Image Processing)

Untuk fitur **Foto Soal & Pembahasan**:
1. **Citra Efemeral:** Foto tugas atau soal yang diambil kamera hanya disimpan sementara di direktori internal privat aplikasi (`context.cacheDir/solver_ephemeral/`).
2. **Pembersihan Otomatis Seketika (Stage 6):** Segera setelah Google ML Kit selesai mengekstrak teks OCR dan model selesai melakukan validasi jawaban, berkas citra asli dihapus secara permanen menggunakan `file.delete()`.
3. **Zero Cloud Upload:** Citra foto soal tidak pernah diunggah ke Supabase Storage atau server cloud mana pun.
4. **Pembersihan Darurat:** Saat aplikasi menerima sinyal `onTrimMemory()` atau ditutup oleh pengguna, modul `EphemeralImageCleaner` menghapus seluruh berkas tersisa di direktori sementara.

---

## 5. Kebijakan Audit & Minimalisasi Logging (Minimal Logging Policy)

Untuk mencegah kebocoran informasi melalui file log (*log leakage*):
- **Dilarang mencatat raw prompt atau isi teks tugas** ke dalam `Logcat`, Android Log, atau file teks lokal.
- Logging hanya diizinkan untuk data metrik performa non-sensitif:
  - Latensi kueri (ms).
  - Jumlah dokumen yang cocok (*match count*).
  - Penggunaan token prompt & completion (angka integer).
  - Kode status error teknis.
- Segala bentuk identitas pribadi (nama mahasiswa, nomor telepon, isi catatan) disaring (*redacted*) sebelum dikirim ke analitik crash sistem (jika ada).
