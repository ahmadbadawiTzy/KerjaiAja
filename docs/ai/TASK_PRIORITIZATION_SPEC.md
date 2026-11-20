# Spesifikasi Perhitungan Prioritas Tugas (Task Prioritization Specification) — KerjaiAja

Dokumen ini mendefinisikan formula matematis deterministik, integrasi penalaran kontekstual AI backend, format transparansi alasan (*explainability*), serta aturan kendali pengguna (*user override*) untuk sistem penentuan prioritas tugas **KerjaiAja**.

---

## 1. Tujuan & Filosofi Penentuan Prioritas

1. **Objektif & Bebas Bias Emosional:** Mengubah penumpukan tugas yang membingungkan menjadi daftar tindakan yang terurut rapi dan rasional.
2. **Explainability Pertama:** Setiap skor prioritas harus dapat dijelaskan alasannya dalam bahasa manusia yang lugas (*"Mengapa tugas ini di atas tugas lainnya?"*).
3. **Kedaulatan Pengguna (User Sovereignty):** Mahasiswa memiliki hak mutlak untuk mengubah urutan prioritas sesuai intuisi atau kondisi nyata di lapangan.
4. **Anti-Guilt Philosophy:** Sistem tidak mempermalukan pengguna ketika tugas terlewat, melainkan segera memberikan jalur pemulihan jadwal.

---

## 2. Formula Prioritas Deterministik (Base Scoring)

Skor dasar prioritas ($S_{base}$) bernilai bilangan bulat antara $0$ hingga $100$, dihitung dengan formula berbobot multi-faktor:

$$S_{base} = \min\left(100, \max\left(0, W_t \cdot T(\Delta t) + W_a \cdot A + W_c \cdot C + W_b \cdot B\right)\right)$$

### 2.1 Bobot Faktor (Weight Distribution)
Total bobot faktor dinormalisasi:
- **$W_t$ (Time to Deadline Weight) = 0.45 (45%)**
- **$W_a$ (Academic Weight Factor) = 0.25 (25%)**
- **$W_c$ (Complexity & Duration Factor) = 0.20 (20%)**
- **$W_b$ (Team Blocking Factor) = 0.10 (10%)**

### 2.2 Fungsi Waktu Menuju Deadline ($T(\Delta t)$)
Di mana $\Delta t$ adalah sisa waktu hingga deadline dalam satuan jam ($t_{due} - t_{now}$):
- $\Delta t \le 0$ (Overdue): $T(\Delta t) = 100$ (masuk ke penanganan khusus anti-guilt)
- $0 < \Delta t \le 12$ jam: $T(\Delta t) = 100 - (\Delta t \cdot 1.5)$
- $12 < \Delta t \le 24$ jam: $T(\Delta t) = 82 - ((\Delta t - 12) \cdot 1.5)$
- $24 < \Delta t \le 72$ jam (1-3 hari): $T(\Delta t) = 64 - ((\Delta t - 24) \cdot 0.5)$
- $72 < \Delta t \le 168$ jam (3-7 hari): $T(\Delta t) = 40 - ((\Delta t - 72) \cdot 0.2)$
- $\Delta t > 168$ jam (> 1 minggu): $T(\Delta t) = \max\left(10, 20 - \frac{\Delta t - 168}{24}\right)$

### 2.3 Komponen Bobot Akademik ($A$)
Berdasarkan persentase kontribusi tugas terhadap nilai akhir semester ($P_{academic} \in [1, 100]$):
$$A = \min(100, P_{academic} \cdot 2.5)$$
*(Tugas dengan bobot 40% langsung mendapat skor faktor $A = 100$).*

### 2.4 Komponen Kompleksitas & Durasi ($C$)
Memperhitungkan estimasi menit kerja ($E$) dan tingkat kompleksitas:
- Multiplier kompleksitas: `LOW` = 0.8, `MEDIUM` = 1.0, `HIGH` = 1.3
- $C = \min\left(100, \left(\frac{E}{180}\right) \cdot 70 \cdot \text{multiplier}\right)$

### 2.5 Komponen Hambatan Tim ($B$)
- Jika tugas adalah bagian dari proyek kelompok dan menjadi prasyarat (*blocking dependency*) bagi pengerjaan anggota lain: $B = 100$.
- Jika tugas kelompok biasa tanpa blocking: $B = 40$.
- Jika tugas mandiri perseorangan: $B = 0$.

---

## 3. Pengayaan Penalaran Kontekstual AI (AI Contextual Enrichment)

Setelah skor deterministik $S_{base}$ dihitung, Go backend memanggil modul AI untuk menganalisis instruksi deskripsi tugas teks bebas:

1. **Deteksi Hambatan Tersembunyi (Hidden Friction Detection):**
   AI memindai apakah instruksi mensyaratkan:
   - Wawancara narasumber / survei kuesioner ke publik ($\to$ membutuhkan lead time tambahan).
   - Percobaan lab basah atau peminjaman alat inventaris.
   - Pembelian material fisik atau perakitan hardware.
2. **Penyesuaian Skor Terikat (*Bounded Adjustment*):**
   AI hanya diizinkan menyesuaikan skor maksimum $\pm 15$ poin:
   $$S_{final} = \min\left(100, \max\left(0, S_{base} + \Delta S_{ai}\right)\right)$$
   dengan $-15 \le \Delta S_{ai} \le +15$.
3. **Penyertaan Alasan Kontekstual:** AI wajib menyertakan alasan tertulis jika melakukan penyesuaian skor.

---

## 4. Klasifikasi Empat Kategori Urgensi

| Kategori Urgensi | Rentang Skor | Warna Neo-Bauhaus | Implikasi Tindakan Mahasiswa |
| :--- | :---: | :---: | :--- |
| **CRITICAL** | $80 - 100$ | Deep Red (`#DC2626`) | Wajib dikerjakan hari ini; tenggat waktu mendesak atau bobot nilai besar. |
| **HIGH** | $60 - 79$ | Bright Coral (`#EA580C`) | Membutuhkan alokasi waktu substansial dalam 48 jam ke depan. |
| **MEDIUM** | $40 - 59$ | Bauhaus Amber (`#D97706`) | Tugas dalam ritme normal (jadwal pengerjaan 3–6 hari ke depan). |
| **LOW** | $0 - 39$ | Slate Blue (`#64748B`) | Tugas dengan tenggat waktu longgar atau tugas bacaan santai. |

---

## 5. Kontrak Penjelasan Transparan (Explainability Rationale)

Setiap entitas tugas yang dikembalikan oleh API memiliki atribut `priority_explanation`:

```json
{
  "priority_level": "CRITICAL",
  "priority_score": 88,
  "factors": [
    { "label": "Waktu Tersisa", "value": "18 jam hingga batas pengumpulan", "impact": "+42 pts" },
    { "label": "Bobot Akademik", "value": "Kontribusi 25% ke nilai akhir", "impact": "+20 pts" },
    { "label": "Ketergantungan Tim", "value": "Menjadi prasyarat pengerjaan Bab 4 oleh Budi", "impact": "+10 pts" },
    { "label": "Analisis AI", "value": "Instruksi memerlukan verifikasi data lab yang memakan waktu", "impact": "+16 pts" }
  ],
  "summary_text": "Tugas ini sangat mendesak karena deadline tersisa kurang dari 24 jam dan pengerjaannya menghambat anggota kelompok Anda."
}
```

---

## 6. Hak Kendali Pengguna (User Priority Override) & Anti-Guilt Rules

1. **Manual Override:**
   Pengguna dapat mengubah prioritas tugas secara manual melalui opsi dropdown. Jika diubah manual:
   - Nilai `is_priority_overridden` menjadi `true`.
   - Formula otomatis tidak akan menimpa pilihan pengguna sampai pengguna menekan *"Kembalikan ke Rekomendasi Otomatis"*.
2. **Protokol Tugas Terlambat (Overdue Anti-Guilt UX):**
   Jika $t_{now} > t_{due}$ dan tugas belum selesai:
   - Dilarang membunyikan alarm terus-menerus atau menampilkan dialog yang menimbulkan kecemasan.
   - Sistem menampilkan badge netral dengan tiga opsi aksi langsung:
     1. **"Jadwalkan Ulang":** Memilih tanggal baru berdasarkan slot luang jadwal kuliah.
     2. **"Pecah ke Langkah 15 Menit":** Membantu memulai kembali (*kickstart inertia*) dengan sub-langkah kecil.
     3. **"Tandai Selesai / Arsip":** Jika tugas sudah dikumpulkan di portal kampus tanpa sempat dicatat di aplikasi.
