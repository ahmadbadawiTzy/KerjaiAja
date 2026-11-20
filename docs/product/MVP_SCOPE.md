# Ruang Lingkup MVP (Minimum Viable Product Scope) — KerjaiAja

Dokumen ini menetapkan batasan ketat mengenai fitur yang masuk dalam cakupan rilis awal (MVP) versus fitur yang ditunda ke rilis iteratif berikutnya.

---

## 1. Prinsip Penentuan Ruang Lingkup (Scoping Principles)

1. **Local-First Independence:** Fungsionalitas inti aplikasi harus 100% dapat digunakan tanpa memerlukan koneksi internet maupun ketersediaan mesin AI lokal.
2. **Low Surface Area, High Reliability:** Lebih mengutamakan kestabilan alur pengerjaan tugas akademik harian dibanding kuantitas fitur periferal.
3. **Decoupled Intelligence:** Fitur AI diposisikan sebagai akselerator produktivitas (*enhancement layer*), bukan prasyarat fungsional (*hard dependency*).

---

## 2. Matriks Ruang Lingkup Fitur (Feature Scope Matrix)

| Modul / Kapabilitas | Status MVP (v1.0) | Pasca-MVP (v1.x / v2.0) | Justifikasi |
| :--- | :---: | :---: | :--- |
| **Academic Task CRUD** | ✅ In-Scope | - | Fondasi utama pelacakan tugas akademik harian. |
| **Subject Management** | ✅ In-Scope | - | Pengelompokan tugas per mata kuliah/dosen. |
| **Assignment Breakdown Manual** | ✅ In-Scope | - | Memecah tugas menjadi sub-langkah secara manual. |
| **Study Planning & Focus Timer** | ✅ In-Scope | - | Pengatur waktu fokus terintegrasi dengan tugas. |
| **Submission Checklist** | ✅ In-Scope | - | Mencegah kekeliruan pengumpulan berkas tugas. |
| **Group Project Tracking (Lokal)** | ✅ In-Scope | - | Pembagian tanggung jawab tugas tim pada database lokal. |
| **Adaptive Local AI Breakdown** | 🟡 In-Scope (Eksperimental) | - | Bantuan dekonstruksi instruksi tugas via Gemma 3 1B IT. |
| **On-Demand Model Download & SHA-256**| 🟡 In-Scope | - | Menjaga ukuran APK awal tetap ramping. |
| **Cloud Sync & Auth (Supabase)** | ❌ Out-of-Scope (Fase Lanjutan) | ✅ Rencana v1.2 | Fokus awal diselesaikan pada keandalan basis data lokal (Room). |
| **Realtime Group Collaboration** | ❌ Out-of-Scope (Fase Lanjutan) | ✅ Rencana v1.3 | Membutuhkan sinkronisasi jaringan yang kompleks; MVP fokus lokal. |
| **OCR Silabus / Pindai Dokumen** | ❌ Out-of-Scope | ✅ Rencana v2.0 | Kompleksitas preprocessing gambar ditunda hingga teks pipeline stabil. |
| **Multi-platform / Web Portal** | ❌ Out-of-Scope | ❌ Di luar roadmap | KerjaiAja difokuskan secara eksklusif sebagai Android Native. |

---

## 3. Kriteria Keberhasilan MVP (MVP Success Criteria)

1. Pelajar dapat mencatat tugas baru, mengaitkannya ke mata kuliah, dan menetapkan tenggat waktu dalam waktu < 20 detik.
2. Pelajar dapat memecah tugas besar menjadi minimal 3 sub-tugas yang terorganisasi dan memantau status penyelesaiannya.
3. Fitur pengingat dan checklist pengumpulan berhasil mencegah kesalahan format berkas sebelum pengumpulan.
4. Pada perangkat yang kompatibel, model AI lokal berhasil diunduh secara mandiri, diverifikasi SHA-256 integritasnya, dan mampu memberikan rekomendasi sub-langkah tugas tanpa kendala crash memori.
5. Pada perangkat yang tidak mendukung AI (RAM terbatas atau prosesor lama), aplikasi tetap beroperasi secara mulus dengan mode manual tanpa memunculkan error atau tampilan yang terblokir.
