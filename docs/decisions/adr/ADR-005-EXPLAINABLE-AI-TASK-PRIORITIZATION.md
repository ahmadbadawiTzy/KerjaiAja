# ADR-005: Explainable AI Task Prioritization Engine

- **Status:** ACCEPTED
- **Tanggal:** 2026-11-20
- **Pengambil Keputusan:** Tim Arsitektur KerjaiAja (Senior Product Architect & AI Engineer)
- **Terkait Dokumen:** [`docs/ai/TASK_PRIORITIZATION_SPEC.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/ai/TASK_PRIORITIZATION_SPEC.md), [`docs/product/PRODUCT_REQUIREMENTS.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/product/PRODUCT_REQUIREMENTS.md)

---

## 1. Konteks (Context)
Aplikasi manajemen tugas tradisional sering kali membebani mahasiswa dengan daftar to-do panjang tanpa arah yang jelas tugas mana yang harus diselesaikan terlebih dahulu. Sebagian besar aplikasi hanya mengurutkan berdasarkan tanggal tenggat waktu terdekat (*closest due date*), yang sering kali keliru karena mengabaikan estimasi durasi, tingkat kesulitan, dampak bobot nilai akademik, dependensi antar-tugas, dan ketersediaan waktu luang pengguna.

## 2. Keputusan (Decision)
Membangun sistem penentuan prioritas hibrida **Explainable AI Task Prioritization Engine**:
1. **Pendekatan Hibrida (Deterministic + Contextual AI):**
   - **Aturan Deterministik:** Menghitung skor dasar numerik (*urgency score*) berdasarkan parameter matematis terukur: waktu tersisa hingga deadline, estimasi waktu kerja, bobot mata kuliah, dan status dependensi prasyarat.
   - **Contextual AI Reasoning:** Menganalisis instruksi tugas, tingkat ambiguitas, dan beban kerja harian mahasiswa untuk memberikan penyesuaian skor konteks (*context modifier*).
2. **Kategori Urgensi Terkalibrasi:**
   - **`CRITICAL` (Skor >= 85):** Tugas berbobot tinggi dengan deadline < 24–48 jam atau memblokir tugas lain. Tindakan: Segera dikerjakan hari ini.
   - **`HIGH` (Skor 65–84):** Tugas besar yang harus mulai dicicil agar tidak menumpuk mendekati batas akhir. Tindakan: Dijadwalkan ke sesi fokus.
   - **`MEDIUM` (Skor 40–64):** Tugas rutin dengan batas waktu aman. Tindakan: Pengerjaan terencana.
   - **`LOW` (Skor < 40):** Tugas fleksibel atau pengerjaan mandiri tanpa risiko akademik langsung.
3. **Prinsip Transparansi & Dapat Dijelaskan (*Explainability*):**
   - Setiap kartu tugas tidak hanya menampilkan label prioritas, tetapi menyertakan kartu penjelasan singkat (*reasoning chips*), misalnya: *"Prioritas Tinggi karena deadline tersisa 2 hari dan membutuhkan estimasi 4 jam pengerjaan."*
4. **User Override (Kontrol Penuh Pengguna):**
   - Pengguna memiliki hak mutlak untuk membatalkan atau mengubah level prioritas hasil kalkulasi AI. Sistem mencatat override ini untuk menyesuaikan preferensi berikutnya.
5. **Recalculation Dinamis:**
   - Skor dihitung ulang otomatis saat terjadi pembaruan: deadline diedit, sub-tugas dicentang, atau sesi hari baru dimulai.

## 3. Alternatif yang Dipertimbangkan (Alternatives Considered)
- **Urutan Deadline Murni (Sort by Due Date):** Ditolak karena mengabaikan kompleksitas tugas; tugas makalah 20 halaman yang deadline-nya 3 hari lagi akan kalah urutan dari kuis online 5 menit yang deadline-nya besok pagi.
- **Blackbox AI Ranking (Peringkat AI Murni Tanpa Alasan):** Ditolak karena pengguna tidak akan percaya pada rekomendasi jika tidak memahami alasan logis di baliknya.

## 4. Konsekuensi (Consequences)
- **Positif:** Mengurangi beban kognitif dan *analysis paralysis* pada mahasiswa secara dramatis; memberikan rasa kendali dan arah kerja yang terstruktur.
- **Negatif:** Membutuhkan penyesuaian threshold skor secara berkala berdasarkan pengujian empiris.
- **Implikasi Biaya:** Ringan; perhitungan deterministik berjalan lokal di Go backend/cache Android, sedangkan AI context modifier dipanggil secara efisien saat pembuatan/pembaruan tugas.
