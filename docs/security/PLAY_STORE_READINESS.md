# Kesiapan & Kepatuhan Google Play Store (Play Store Readiness & Security) — KerjaiAja
## Target SDK 35 · Play Store Data Safety · Account Deletion Compliance · Minimal Permissions

Dokumen ini mendefinisikan standar keamanan, kepatuhan teknis, dan formulir deklarasi **Google Play Store Policy** untuk aplikasi **KerjaiAja** pasca-rebaseline arsitektur hybrid, diselaraskan dengan [ADR-011](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-011-PLAY-STORE-SECURITY-COMPLIANCE.md), [ADR-020](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-020-SUPABASE-MANAGED-BAAS.md), dan standar **OWASP MASVS**.

---

## 1. Konfigurasi SDK & Toolchain Android

| Parameter | Nilai Konfigurasi | Keterangan Kepatuhan |
| :--- | :---: | :--- |
| **`compileSdk`** | `35` | Android 15 (Vanilla Ice Cream) SDK terbaru. |
| **`targetSdk`** | `35` | Wajib untuk seluruh rilis aplikasi baru di Play Store. |
| **`minSdk`** | `26` | Android 8.0 (Oreo), mencakup $\ge 95\%$ perangkat aktif global. |
| **Distribution Format** | `.aab` | Android App Bundle dengan Play App Signing (ukuran awal $\le 20\text{ MB}$). |
| **Code Shrinking & Obfuscation** | R8 / ProGuard Enabled | Memperkecil APK dan melindungi kode sumber dari rekayasa balik (*reverse engineering*). |

---

## 2. Prinsip Minimisasi Izin (Permission Minimization)

Aplikasi menerapkan pembatasan izin ketat dalam `AndroidManifest.xml`:

### 2.1 Izin yang Digunakan (Permitted Permissions)
- `android.permission.INTERNET`: Digunakan untuk sinkronisasi cloud Supabase (PostgREST, Realtime, Storage) dan mengunduh bobot model Google Gemma on-demand. **DILARANG MENGIRIM DATA PROMPT AI KE SERVER EKSTERNAL.**
- `android.permission.ACCESS_NETWORK_STATE`: Untuk mendeteksi koneksi jaringan sebelum memulai sinkronisasi background atau unduhan bobot model AI.
- `android.permission.POST_NOTIFICATIONS`: Meminta izin runtime pada Android 13+ (API 33+) untuk alarm pengingat tugas.
- `android.permission.SCHEDULE_EXACT_ALARM`: Digunakan secara terukur untuk membunyikan alarm pengingat tenggat waktu akademik berpresisi tinggi.

### 2.2 Izin yang DILARANG KERAS (Strictly Forbidden Permissions)
- ❌ `READ_CONTACTS` / `WRITE_CONTACTS` (Zero contact access).
- ❌ `ACCESS_FINE_LOCATION` / `ACCESS_COARSE_LOCATION` (Zero geolocation tracking).
- ❌ `READ_SMS` / `RECEIVE_SMS` (Tidak ada SMS).
- ❌ `CAMERA` / `RECORD_AUDIO` (Tidak ada pemrosesan sensor kamera/mikrofon pada MVP).
- ❌ `READ_EXTERNAL_STORAGE` / `MANAGE_EXTERNAL_STORAGE` (Menggunakan Scoped Storage privat aplikasi).

---

## 3. Pemenuhan Regulasi Google Play Data Safety

Tabel pemetaan deklarasi Google Play Data Safety Section:

| Kategori Data | Jenis Data | Dikumpulkan ke Server? | Dibagikan ke Pihak Ketiga? | Penjelasan Teknis |
| :--- | :--- | :---: | :---: | :--- |
| **Personal Info** | Email & Nama | Opsional (Jika Login) | **TIDAK** | Disimpan di Supabase Auth & Profiles terenkripsi; Guest mode 100% lokal. |
| **User Activity** | Tugas, mata kuliah, jadwal | Opsional (Jika Login) | **TIDAK** | Disinkronkan ke Supabase PostgreSQL terenkripsi via HTTPS TLS 1.3. |
| **AI Prompts & Queries** | Teks analisis tugas | **TIDAK** | **TIDAK** | Diproses 100% secara on-device oleh Google Gemma via LiteRT-LM. |
| **File Attachments** | Dokumen tugas (PDF/DOCX) | Opsional (Jika Login) | **TIDAK** | Disimpan di private bucket Supabase Storage dengan akses RLS. |

### Deklarasi Resmi Google Play Data Safety:
> - **Data is encrypted in transit:** Seluruh komunikasi jaringan cloud menggunakan HTTPS / TLS 1.3.
> - **No data shared with third parties:** Data pengguna tidak pernah dijual atau dibagikan ke pihak ketiga atau pengiklan.
> - **Users can request data deletion:** Pengguna memiliki kendali penuh untuk menghapus data.

---

## 4. Kepatuhan Kebijakan Penghapusan Akun (Account Deletion Policy)

Sesuai regulasi wajib Google Play Store:
1. **In-App Account Deletion:** Menu Pengaturan menyediakan opsi *"Hapus Akun & Seluruh Data"*.
2. **Cascading Deletion di Supabase:** Memanggil fungsi database PostgreSQL yang menghapus baris pengguna dari `auth.users`, yang secara otomatis menghapus seluruh entitas di tabel `profiles`, `tasks`, `subtasks`, `team_members`, serta file di Supabase Storage (`ON DELETE CASCADE`).
3. **Web-Based Deletion URL:** Disediakan halaman web publik sederhana (`https://kerjaiaja.web.app/delete-account` atau GitHub Pages) yang memungkinkan pengguna meminta penghapusan akun tanpa harus menginstal ulang aplikasi.
4. **Hapus Bobot Model AI:** Pengguna dapat menghapus bobot model AI lokal kapan saja untuk membebaskan ruang disk ponsel.
