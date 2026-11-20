# Spesifikasi Kebutuhan Pengetahuan Akademik (Academic Knowledge)
## Cakupan 12 Rumpun Mata Pelajaran, Sitasi Terverifikasi, dan Kebutuhan Produk

- **Status:** APPROVED & LOCKED
- **Lokasi Dokumen:** `docs/product/ACADEMIC_KNOWLEDGE_REQUIREMENTS.md`
- **Terkait:** [`KNOWLEDGE_INTELLIGENCE_LAYER.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/KNOWLEDGE_INTELLIGENCE_LAYER.md), [`PHOTO_QUESTION_SOLVER_SPECIFICATION.md`](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/ai/PHOTO_QUESTION_SOLVER_SPECIFICATION.md)

---

## 1. Visi Produk Pengetahuan Akademik

Fitur **Foto Soal & Pembahasan** didukung oleh basis pengetahuan akademik yang terkurasi untuk memastikan mahasiswa mendapatkan pembahasan yang **berbobot secara pedagogis, faktual, dan memiliki rujukan ilmiah yang jelas**. Sistem dilarang bertindak seperti chatbot halusinatif yang menjawab tanpa landasan teori.

Prinsip Produk:
1. **Curated & Authoritative:** Materi rujukan bersumber dari buku teks standar sekolah menengah/perguruan tinggi (misal: Kemendikbudristek, kurikulum nasional, buku ajar standar).
2. **Transparent Grounding:** Pengguna dapat melihat buku referensi, bab materi, dan halaman rujukan yang digunakan sistem untuk menjawab soal.
3. **Strict Non-Hallucination:** Jika materi pendukung tidak ditemukan di basis pengetahuan lokal, sistem wajib memberi tahu pengguna secara jujur bahwa pembahasan dihasilkan murni dari penalaran dasar model tanpa konfirmasi buku rujukan lokal.

---

## 2. Pemetaan Kebutuhan 12 Rumpun Mata Pelajaran

| Rumpun Ilmu | Topik Utama yang Didukung | Kebutuhan Pengetahuan Spesifik | Standar Validasi |
| :--- | :--- | :--- | :--- |
| **1. Matematika** | Aljabar, Geometri, Trigonometri, Kalkulus, Statistika, Peluang | Rumus baku, teorema (misal: Phytagoras, Turunan), langkah eliminasi/substitusi, sifat logaritma. | Wajib menyertakan rumus dasar sebelum melakukan substitusi angka. |
| **2. Fisika** | Mekanika Newton, Usaha & Energi, Termodinamika, Gelombang, Optik, Listrik Dinamis | Hukum fisika, formula matematis, konstanta standar ($g = 9.8\text{ m/s}^2$ atau $10\text{ m/s}^2$), konversi satuan SI. | Validasi ketat satuan SI pada jawaban akhir (Joule, Newton, Pascal, dsb). |
| **3. Kimia** | Stoikiometri, Struktur Atom, Tabel Periodik, Larutan Asam-Basa, Termokimia, Redoks | Nomor massa/atom, rumus molekul, persamaan reaksi setara, hukum kekekalan massa. | Persamaan reaksi wajib setara (jumlah atom kiri = kanan). |
| **4. Biologi** | Struktur Sel, Metabolisme, Genetika Mendel, Ekologi, Anatomi Fisiologi Manusia | Tahapan proses biologis (misal: Glikolisis, Meiosis), fungsi organel sel, hukum persilangan. | Alur tahapan proses harus kronologis dan berurutan. |
| **5. Bahasa Indonesia**| PUEBI/EYD, Struktur Teks (Eksposisi, Narasi, Argumentasi), Majas, Paragraf Induktif/Deduktif | Kaidah ejaan baku KBBI, kaidah kalimat efektif, jenis konjungsi, ide pokok. | Referensi kaidah ejaan resmi (PUEBI/EYD V). |
| **6. Bahasa Inggris** | Tenses, Passive Voice, Conditional Sentences, Reading Comprehension, Direct/Indirect Speech | Rumus gramatikal tenses, pola kalimat, penanda waktu (*time signals*). | Konsistensi tenses dan rumus pola kalimat bahasa Inggris. |
| **7. Sejarah** | Sejarah Nasional Indonesia, Masa Kolonial, Pergerakan Nasional, Sejarah Dunia | Kronologi tahun, peristiwa penting, tokoh sejarah, latar belakang dan dampak peristiwa. | Urutan linimasa sejarah tidak boleh terbalik. |
| **8. Geografi** | Litosfer, Atmosfer, Hidrosfer, Dinamika Kependudukan, SIG & Pemetaan | Karakteristik lapisan bumi, tipe iklim (Koppen/Schmidt), rumus perhitungan densitas penduduk. | Kesesuaian terminologi geografis standar. |
| **9. Ekonomi** | Permintaan & Penawaran, Keseimbangan Pasar, Akuntansi Dasar, Kebijakan Fiskal/Moneter | Rumus fungsi permintaan/penawaran ($Q_d = Q_s$), rumus elastisitas, persamaan dasar akuntansi. | Keseimbangan debit-kredit pada pencatatan akuntansi. |
| **10. Sosiologi** | Struktur Sosial, Diferensiasi, Interaksi Sosial, Konflik & Integrasi, Perubahan Sosial | Konsep sosiologis dari tokoh utama (Durkheim, Weber, Marx), teori interaksi. | Kesesuaian definisi konsep sosiologis terverifikasi. |
| **11. PPKn** | Nilai-Nilai Pancasila, UUD 1945, Hak Asasi Manusia, Sistem Pemerintahan Indonesia | Pasal-pasal UUD 1945, tata urutan perundang-undangan nasional, makna sila Pancasila. | Nomor pasal dan naskah hukum wajib akurat tanpa distorsi. |
| **12. Terapan / TIK** | Algoritma Dasar, Logika Boolean, Jaringan Komputer Dasar, Representasi Data | Konsep variabel, gerbang logika (AND, OR, NOT), representasi biner, topologi jaringan. | Logika matematika biner dan tabel kebenaran valid. |

---

## 3. Kebutuhan Penelusuran Sumber & Sitasi (Source Traceability)

Saat pengguna membuka solusi pembahasan foto soal, UI menampilkan tombol **"Rujukan Materi Terverifikasi"**:
- **Format Tampilan Sitasi:**
  - **Nama Dokumen / Buku:** Misal: *"Buku Siswa Fisika Kelas XI Kurikulum Merdeka"*.
  - **Bab / Topik:** *"Bab 4: Fluida Dinamis"*.
  - **Halaman Rujukan:** *"Halaman 87–90"*.
  - **Kutipan Kunci (*Quoted Snippet*):** Kutipan teks 1–2 kalimat dari buku referensi yang menjadi dasar solusi.
- **Larangan Halusinasi Sitasi:**
  - Sistem dilarang mengarang nomor halaman palsu atau nama pengarang fiktif. Jika metadata halaman tidak tersedia di berkas lokal, tampilkan judul bab dan nama dokumen saja.

---

## 4. Alur Interaksi Pengguna (User Journey)

```
[Foto Soal Diambil]
       |
       v
[Layar Peninjauan OCR (Human-in-the-Loop)]
       |  (Pengguna dapat mengedit teks soal & memilih mata pelajaran)
       v
[Retrieval Pengetahuan Akademik Otomatis]
       |  (Pencarian leksikal di academic_fts lokal)
       v
[Inferensi Model On-Device + Validasi Jawaban 7 Bagian]
       |
       v
[Layar Solusi Pembahasan]
  - Bagian 1-6: Pembahasan Terstruktur Lengkap
  - Bagian 7: "Rujukan & Dasar Teori" (Expandable Card)
      -> Menampilkan dokumen sumber, bab, halaman, dan kutipan konsep
```
