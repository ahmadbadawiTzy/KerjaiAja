# Sistem Desain Neo-Bauhaus (Neo-Bauhaus Design System) — KerjaiAja

Dokumen ini mendefinisikan sistem desain lengkap **KerjaiAja** berbasis filosofi **Neo-Bauhaus × Contemporary Gen Z × Professional Academic Productivity**, mengacu pada panduan resmi skill `.agents/skills/bauhaus-design/` dan standar Jetpack Compose Material 3.

---

## 1. Filosofi Inti: Form Follows Function

1. **Kejelasan Fungsional Sebelum Dekorasi:**  
   Setiap garis pembatas (*border*), kontainer warna, dan ukuran teks memiliki peran konkret untuk mengomunikasikan prioritas akademik, status pengerjaan, atau tindakan berikutnya. Tidak ada elemen visual yang ada hanya untuk "pemanis" tanpa nilai guna.
2. **Geometri Berani & Terstruktur (Bold Geometry):**  
   Penggunaan bentuk persegi, lingkaran, dan garis tegas dengan kontur berbobot (*crisp 1.5dp - 2dp high-contrast borders*).
3. **Anti-Guilt & Non-Judgemental UX:**  
   Sistem tidak menggunakan warna alarm histeris atau modal dialog yang mempermalukan mahasiswa ketika tugas terlambat. Sebaliknya, antarmuka menyajikan opsi aksi adaptif yang tenang dan terarah (*constructive recovery*).
4. **Kepatuhan Aksesibilitas WCAG 2.1 AA:**  
   Rasio kontras warna teks terhadap latar belakang minimal 4.5:1 untuk teks normal dan 3.0:1 untuk teks besar/elemen antarmuka grafis.

---

## 2. Palet Warna & Token Semantik (Color Palette Tokens)

### 2.1 Warna Kanvas & Netral
| Token Name | Hex Light | Hex Dark | Penggunaan |
| :--- | :---: | :---: | :--- |
| `canvas_background` | `#F8F9FA` | `#121316` | Latar belakang layar utama (warm off-white / deep charcoal) |
| `surface_card` | `#FFFFFF` | `#1E2024` | Latar kartu tugas dan kontainer komponen |
| `surface_border` | `#1E293B` | `#475569` | Garis tepi geometris tegas (1.5dp - 2dp) |
| `text_primary` | `#0F172A` | `#F8FAFC` | Judul tugas, teks headline, angka countdown |
| `text_secondary` | `#475569` | `#94A3B8` | Subtitle, nama dosen, keterangan instruksi |
| `text_muted` | `#64748B` | `#64748B` | Timestamp, metadata tanggal dibuat |

### 2.2 Warna Semantik Prioritas & Status Akademik
| Tingkat Urgensi | Hex Light | Hex Dark | Makna Fungsional |
| :--- | :---: | :---: | :--- |
| **CRITICAL** | `#DC2626` | `#EF4444` | Bauhaus Crimson: Deadline < 24 jam atau bobot $\ge 30\%$ |
| **HIGH** | `#EA580C` | `#F97316` | Bauhaus Coral: Deadline 2–3 hari, butuh aksi segera |
| **MEDIUM** | `#D97706` | `#F59E0B` | Bauhaus Amber: Tugas normal terjadwal (4–7 hari) |
| **LOW** | `#2563EB` | `#60A5FA` | Bauhaus Cobalt: Bacaan mandiri / deadline longgar |
| **COMPLETED** | `#059669` | `#10B981` | Bauhaus Emerald: Tugas selesai & diverifikasi |

---

## 3. Sistem Tipografi (Inter Font Family)

Menggunakan keluarga font **Inter** (Google Fonts / SIL Open Font License) dengan penekanan pada keterbacaan numerik tabular (*tabular figures*):

| Kategori Tipe | Font Weight | Ukuran (sp) | Line Height (sp) | Tracking | Penggunaan |
| :--- | :---: | :---: | :---: | :---: | :--- |
| `Display Large` | Bold (700) | 32 | 40 | -0.5 | Skor Prioritas Dashboard & Countdown Timer |
| `Headline Medium`| Bold (700) | 24 | 32 | -0.2 | Judul Halaman Utama |
| `Title Large` | SemiBold (600) | 20 | 28 | 0.0 | Judul Tugas di Kartu Detail |
| `Title Medium` | Medium (500) | 16 | 24 | +0.1 | Judul Tugas di Task List Feed |
| `Body Large` | Regular (400) | 16 | 24 | +0.2 | Teks Deskripsi Instruksi Tugas |
| `Body Medium` | Regular (400) | 14 | 20 | +0.2 | Detail Sub-tugas & Langkah Belajar |
| `Label Large` | Bold (700) | 12 | 16 | +0.5 | Badge Kategori Urgensi & Tombol Aksi |
| `Label Medium` | Medium (500) | 11 | 14 | +0.4 | Tag Mata Kuliah & Role Anggota Tim |

---

## 4. Spacing, Grid & Geometri

### 4.1 Skala Spacing Berbasis 4dp
- `space_xxs`: 4dp (padding internal badge)
- `space_xs`: 8dp (gap antar elemen mikro / tag)
- `space_sm`: 12dp (padding internal kartu ringkas)
- `space_md`: 16dp (margin standar layar & padding kartu utama)
- `space_lg`: 24dp (gap antar seksi informasi)
- `space_xl`: 32dp (margin atas headline)

### 4.2 Grid 12-Kolom & Tata Letak
- Klien Android menggunakan grid proporsional matematis:
  - Layar Ponsel Portrait: Margin luar 16dp, gutter 8dp, tata letak satu kolom vertikal.
  - Layar Tablet / Foldable: Margin luar 24dp, 2 kolom terstruktur (Panel Navigasi/Prioritas di kiri, Detail Tugas di kanan).

### 4.3 Radius Sudut Fungsional (Corner Radii)
- `radius_none`: 0dp (tabel data & divider garis)
- `radius_sm`: 6dp (input text field & button sekunder)
- `radius_md`: 12dp (kartu tugas utama & kontainer tim)
- `radius_pill`: 999dp (badge status, tombol aksi cepat, chip filter)

---

## 5. Komponen Kunci Neo-Bauhaus di Jetpack Compose

### 5.1 Kartu Tugas Prioritas (`BauhausTaskCard`)
- Kontainer putih/charcoal dengan border tegas `1.5dp` warna `surface_border`.
- Header: Strip warna penanda mata kuliah di sisi kiri (lebar 6dp).
- Body: Judul tugas (`Title Medium`, bold) + badge urgensi bentuk pill di pojok kanan atas.
- Footer: Indikator sub-tugas (misal: `"3/5 Selesai"`) dan batas waktu relatif (`"Tersisa 18 jam"`).
- Opsi Bayangan Geometris: Hard drop shadow `2dp x 2dp` offset tanpa blur.

### 5.2 Pill Urgensi Transparan (`BauhausPriorityPill`)
- Badge bentuk pill dengan warna latar berbobot 15% opacity dan teks solid bold.
- Menyertakan ikon geometris mikro:
  - `CRITICAL`: Segitiga runcing $\blacktriangle$
  - `HIGH`: Persegi solid $\blacksquare$
  - `MEDIUM`: Lingkaran solid $\bullet$
  - `LOW`: Garis horizontal $-$

### 5.3 Anti-Guilt Recovery Banner
- Muncul ketika tugas melewati tenggat waktu:
  - Latar netral warm grey dengan aksen border biru lembut.
  - Teks pembuka: *"Tugas ini belum selesai. Mari kita atur ulang tanpa stres."*
  - Tiga tombol aksi pill: `[Jadwalkan Ulang]`, `[Pecah 15 Menit]`, `[Tandai Selesai]`.
