# ADR-008: Neo-Bauhaus Design Direction & Contemporary Academic Productivity UI

## Status
ACCEPTED (2026-11-20)

## Konteks
Aplikasi produktivitas akademik di pasar saat ini didominasi oleh dua ekstrem visual yang problematis bagi mahasiswa:
1. **Corporate / Enterprise Bloatware:** Terlalu kaku, monokrom kelabu yang membosankan, hirarki visual padat, dan menimbulkan rasa tertekan (*burnout trigger*).
2. **Juvenile Gamification / Pastel Clutter:** Terlalu banyak ilustrasi kartun, animasi berlebihan yang mengalihkan perhatian, dan palet warna pastel pudar dengan kontras rendah yang melanggar standar aksesibilitas WCAG AA.

Mahasiswa Gen Z membutuhkan antarmuka yang:
- Segar, modern, dan memiliki karakter visual kuat (*distinguishable identity*).
- Menghormati fungsi utama: kejelasan status tugas, transparansi prioritas, dan navigasi cepat tanpa distraksi.
- Filosofi UX anti-guilt: tidak menghakimi atau mempermalukan pengguna ketika tugas terlambat, melainkan memberikan rekomendasi aksi adaptif yang jelas.

## Keputusan
KerjaiAja mengadopsi bahasa desain **Neo-Bauhaus × Contemporary Gen Z × Professional Academic Productivity**:
1. **Filosofi Form Follows Function:** Setiap elemen UI (warna, border, kontainer, tipografi) memiliki tujuan fungsional nyata untuk mengomunikasikan prioritas dan status akademik, bukan sekadar dekorasi.
2. **Tipografi & Skala Matematis:** Menggunakan font Inter (open-source) dengan skala tipe matematis tegas, kontras berat huruf (Bold/Black untuk headline vs Regular/Medium untuk data).
3. **Grid & Geometri Berani:** Grid 12-kolom terstruktur, kontainer dengan border geometris tegas (1.5dp - 2dp high-contrast), radius sudut fungsional (8dp - 12dp untuk kartu tugas, 999dp untuk pill status/action).
4. **Palet Warna Fungsional Berkontras Tinggi:**
   - Background: Off-White / Light Cream canvas untuk kenyamanan membaca jangka panjang (`#F8F9FA` / `#F3F4F6`), serta Dark Canvas (`#121316`) untuk mode gelap.
   - Primary: Deep Electric Cobalt Blue (`#1E40AF` / `#2563EB`) melambangkan fokus, reliabilitas, dan ketegasan akademik.
   - Secondary / Accent: Energetic Bauhaus Amber/Yellow (`#F59E0B`) dan Crimson Accent (`#DC2626`) khusus indikator urgensi kritis.
   - Semantic Prioritas: Critical (Bauhaus Red), High (Bauhaus Coral/Orange), Medium (Bauhaus Amber), Low (Bauhaus Slate/Blue-Grey).
5. **Anti-Guilt UX:**
   - Status *overdue* tidak menggunakan warna merah menyala dengan alarm agresif, melainkan badge netral dengan opsi aksi satu sentuhan: *Reschedule*, *Split into Smaller Steps*, atau *Mark Completed*.

## Konsekuensi
### Positif
- Identitas produk sangat khas dan menonjol di Google Play Store tanpa terlihat kekanak-kanakan atau membosankan.
- Keterbacaan dan kejelasan informasi (*information hierarchy*) sangat tinggi pada layar ponsel berbagai ukuran.
- Sepenuhnya mematuhi standar aksesibilitas WCAG 2.1 AA (rasio kontras teks minimal 4.5:1).
- Mudah diimplementasikan secara modular pada Jetpack Compose menggunakan Material 3 kustom theme tokens.

### Negatif / Mitigasi
- Memerlukan pendefinisian token desain Compose yang disiplin agar komponen custom tidak menyimpang dari pedoman.
  - *Mitigasi:* Spesifikasi lengkap dituangkan ke dalam `docs/design/DESIGN_SYSTEM.md` dan diintegrasikan dengan skill `.agents/skills/bauhaus-design/`.
