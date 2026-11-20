# ADR-022: Offline-First Synchronization Architecture with Resilient Conflict Resolution

## Status
Accepted

## Konteks
Aplikasi produktivitas harus responsif secara instan dan tidak boleh terblokir oleh latency jaringan, koneksi lambat, atau ketiadaan sinyal internet (misalnya saat mahasiswa berada di ruang kuliah bawah tanah atau perjalanan). Selain itu, ketika pengguna mengedit tugas dari dua perangkat berbeda dalam keadaan offline, sistem harus mampu mendeteksi dan menyelesaikan konflik data secara andal.

## Keputusan
Kami menetapkan arsitektur **Offline-First Synchronization**:
1. **Local-Write-First via Room Database:**
   - Semua operasi CRUD (Create, Read, Update, Delete) ditulis langsung ke database lokal Room terlebih dahulu. UI segera bereaksi terhadap perubahan data lokal (instant feedback).
2. **Pending Sync Queue (`sync_pending_queue`):**
   - Setiap mutasi lokal dicatat dalam tabel `sync_pending_queue` lokal yang menyimpan tipe operasi (`INSERT`, `UPDATE`, `DELETE`), nama tabel target, record ID, dan timestamp mutasi.
3. **Background Sync Worker via Android WorkManager:**
   - `SyncWorker` berjalan secara terjadwal atau saat konektivitas jaringan terdeteksi (`NetworkType.CONNECTED`).
   - Melakukan batch processing mutasi lokal ke Supabase PostgreSQL via PostgREST client, dan menarik perubahan terbaru dari cloud.
4. **Soft Deletion (Tombstone):**
   - Penghapusan data tidak menghapus baris secara fisik melainkan menyetel kolom `deleted_at = TIMESTAMP`. Ini mencegah data yang dihapus offline muncul kembali saat disinkronkan dengan cloud.
5. **Conflict Resolution Strategy:**
   - Menggunakan field versioning `updated_at` dan monotonic revision counter `version`.
   - Strategi default: **Last-Write-Wins (LWW)** berdasarkan timestamp UTC terverifikasi untuk perubahan non-kritis.
   - Untuk kolaborasi tim: Jika terjadi konflik edit simultan pada deskripsi/tugas yang sama, sistem menyimpan kedua versi atau meminta konfirmasi pengguna (*prompt user conflict modal*) bila perbedaannya signifikan.

## Konsekuensi
### Positif
- Aplikasi tetap 100% cepat dan responsif tanpa hambatan loading jaringan.
- Toleransi tinggi terhadap pemutusan koneksi internet (network partitioning).
- Kehilangan data diminimalisasi dengan pencatatan mutasi lokal persisten.

### Negatif / Mitigasi
- Membutuhkan penanganan logika sinkronisasi yang teliti dan pengujian edge-case (dimitigasi dengan pengujian unit test dan simulasi network delay/offline).
- Overhead penyimpanan lokal untuk tabel `sync_pending_queue` (dibersihkan secara otomatis setelah batch sync sukses dikonfirmasi).
