# ADR-026: Team Collaboration via Supabase Realtime & PostgreSQL Row Level Security (RLS)

## Status
Accepted

## Konteks
Pengerjaan tugas kelompok akademik membutuhkan fitur kolaborasi tim: berbagi daftar tugas kelompok, pembagian tugas kepada anggota kelompok, pelacakan status progres bersama, dan notifikasi perubahan secara instan. Fitur ini membutuhkan mekanisme kontrol akses yang ketat (agar anggota tim hanya dapat melihat dan mengedit data kelompok mereka sendiri) serta latensi sinkronisasi yang rendah.

## Keputusan
Kami mengimplementasikan kolaborasi tim menggunakan infrastruktur bawaan Supabase:

### 1. Keamanan Data Multi-Tenant via PostgreSQL RLS
- Tabel-tabel tim (`teams`, `team_members`, `team_tasks`, `team_comments`) diproteksi dengan kebijakan **Row Level Security (RLS)** bawaan PostgreSQL.
- Akses baca (`SELECT`) dan tulis (`INSERT`, `UPDATE`, `DELETE`) divalidasi langsung oleh mesin database PostgreSQL berdasarkan `auth.uid()` pengguna Supabase yang sedang login.
- Kebijakan RLS memverifikasi apakah `auth.uid()` terdaftar di tabel `team_members` untuk tim yang bersangkutan dengan role yang memadai (`owner`, `admin`, atau `member`).

### 2. Real-Time Broadcast & Change Data Capture (CDC)
- Menggunakan modul `realtime-kt` dari Supabase Kotlin Client.
- Klien Android berlangganan channel Postgres Changes untuk tabel `team_tasks` dengan filter `team_id=eq.{teamId}`.
- Setiap perubahan status tugas (misalnya dari "In Progress" ke "Completed") atau penambahan komentar baru langsung diterima oleh perangkat anggota tim lainnya secara real-time melalui WebSocket Supabase.

### 3. Batasan Operasional Free Tier
- Penggunaan Supabase Realtime dibatasi maksimal 200 koneksi konkuren secara global (sangat aman untuk kapasitas MVP KerjaiAja).
- Realtime channel ditutup secara otomatis saat pengguna berpindah dari layar kolaborasi tim untuk menghemat baterai dan koneksi WebSocket.

## Konsekuensi
### Positif
- Tidak membutuhkan server perantara WebSocket custom (zero custom socket server).
- Keamanan teruji dan tahan manipulasi karena divalidasi langsung di level database engine PostgreSQL, bukan di kode client.
- Pengalaman pengguna kolaboratif yang responsif dan mulus.

### Negatif / Mitigasi
- Membutuhkan penanganan rekoneksi WebSocket otomatis saat sinyal ponsel terputus (ditangani oleh pustaka resmi `realtime-kt`).
