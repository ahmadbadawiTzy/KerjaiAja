# ADR-001: Android Native with Kotlin and Jetpack Compose

- **Status:** ACCEPTED
- **Tanggal:** 2026-11-20
- **Pengambil Keputusan:** Tim Arsitektur KerjaiAja (Senior Platform Architect)
- **Terkait Dokumen:** [`docs/architecture/ANDROID_ARCHITECTURE.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/ANDROID_ARCHITECTURE.md), [`docs/design/DESIGN_SYSTEM.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/design/DESIGN_SYSTEM.md)

---

## 1. Konteks (Context)
KerjaiAja membutuhkan antarmuka mobile yang sangat responsif, fluid, ergonomis untuk pelajar, dan mampu menerapkan sistem desain visual khas (Neo-Bauhaus) secara presisi tanpa hambatan jank rendering. Aplikasi juga memerlukan akses native ke subsistem Android (WorkManager, NotificationManager, Keystore, Room SQLite).

## 2. Keputusan (Decision)
Mengembangkan aplikasi klien secara eksklusif sebagai **Android Native** menggunakan bahasa **Kotlin** dan toolkit deklaratif modern **Jetpack Compose** dengan fondasi Material 3 yang dikustomisasi menjadi identitas desain Neo-Bauhaus.

Pola arsitektur klien:
- **MVVM + Unidirectional Data Flow (UDF):** State mengalir ke bawah via `StateFlow`, event mengalir ke atas.
- **Repository Pattern & Dependency Inversion:** Memisahkan data cache lokal (Room) dan remote data source (Retrofit API).
- **Dependency Injection:** Hilt (`@HiltViewModel`, `@Inject`).
- **Modern Concurrency:** Kotlin Coroutines dan Flow.

## 3. Alternatif yang Dipertimbangkan (Alternatives Considered)
- **Flutter:** Ditolak untuk mempertahankan kedalaman integrasi native Android, kepatuhan Play Store, performa startup optimal, dan fleksibilitas custom canvas rendering Neo-Bauhaus di Jetpack Compose.
- **React Native:** Ditolak karena overhead bridge JavaScript dan potensi inkonsistensi rendering pada perangkat Android kelas menengah.

## 4. Konsekuensi (Consequences)
- **Positif:** Performa rendering 60/120 FPS tanpa jank, ukuran APK awal ramping, integrasi mulus dengan fitur OS (Dynamic Color, Notification permission, WorkManager).
- **Negatif:** Pengembangan terbatas pada platform Android; aplikasi iOS memerlukan rekayasa terpisah di masa depan.
- **Implikasi Keamanan:** Mengikuti standar OWASP MASVS; tidak ada hardcoded secret pada APK; token disimpan aman di EncryptedSharedPreferences / Android Keystore.
- **Implikasi Biaya:** Zero cost untuk SDK; Android Studio dan toolchain build sepenuhnya gratis.
