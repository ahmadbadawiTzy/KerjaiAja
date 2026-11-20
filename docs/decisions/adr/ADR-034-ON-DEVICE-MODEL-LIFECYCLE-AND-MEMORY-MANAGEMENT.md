# ADR-034: On-Device Model Lifecycle and Memory Management

## Status
ACCEPTED (Disetujui) — 2026-11-20

## Konteks
Menjalankan model AI on-device (seperti Gemma 3 1B IT atau Gemma 3n E2B) pada perangkat Android menghadirkan risiko besar terhadap stabilitas memori dan konsumsi daya. Jika model dimuat secara dini saat aplikasi start (*cold start*), waktu peluncuran aplikasi akan lambat dan menguras RAM latar belakang. Jika model dibiarkan menetap di RAM saat tidak digunakan, sistem operasi Android akan memicu *LowMemoryKiller (LMK)* untuk mematikan aplikasi KerjaiAja atau aplikasi penting lainnya. Selain itu, eksekusi paralel antara OCR dan LLM dapat melipatgandakan alokasi RAM puncak (*peak RAM spike*).

Diperlukan mekanisme manajemen siklus hidup model yang ketat, adaptif terhadap tekanan memori, dan hemat daya.

## Keputusan
Menetapkan kebijakan dan arsitektur **Model Lifecycle and Memory Management** terpadu:
1. **Lazy Loading Mutlak:** Model bobot AI dilarang dimuat saat aplikasi start. Cold start aplikasi harus $< 800\text{ ms}$ dengan konsumsi memori dasar $\le 80\text{ MB}$. Model hanya diinisialisasi on-demand saat layar Asisten atau Foto Soal dibuka.
2. **Sesi Inferensi Tunggal (*Single Active Session*):** Dilarang menjalankan inferensi paralel. OCR dan inferensi LLM dieksekusi secara sekuensial. Permintaan inferensi baru ditolak atau diantrekan jika sesi lain sedang aktif.
3. **Timeout & Pembatalan Anggun (*Graceful Cancellation*):** Inferensi dilindungi batas waktu maksimal 15 detik. Jika pengguna membatalkan atau bernavigasi keluar, pekerjaan inferensi dibatalkan seketika (`cancelChildren()`) dan memori token buffer dibebaskan.
4. **Auto-Unload 5 Menit:** Jika tidak ada interaksi Asisten atau Foto Soal selama 5 menit berturut-turut, runtime model otomatis di-unload dari RAM, mengembalikan alokasi memori ke baseline ~60–80 MB.
5. **Respon Tekanan Memori (*Memory Pressure Callback*):** Mendengarkan `ComponentCallbacks2.onTrimMemory()`. Jika status mencapai `TRIM_MEMORY_RUNNING_CRITICAL`, runtime model segera di-unload secara paksa dan `System.gc()` dipanggil.
6. **Kesadaran Termal & Baterai:** Memantau `PowerManager.OnThermalStatusChangedListener` dan level baterai. Jika status termal `SEVERE` atau level baterai $\le 15\%$ (saat tidak di-charge), inferensi dinonaktifkan sementara demi keselamatan perangkat.

## Konsekuensi
- **Positif:**
  - Waktu cold start aplikasi tetap instan dan konsisten.
  - Zero LMK crash saat aplikasi berada di latar belakang.
  - Pengurangan suhu perangkat dan pencegahan thermal throttling.
  - Alokasi RAM selalu terkendali dalam batas aman perangkat mid-range.
- **Negatif / Tantangan:**
  - Terdapat jeda inisialisasi (~1.5–2.5 detik) saat pertama kali pengguna membuka fitur AI setelah periode idle.
  - Perlu penanganan UI transisi status yang informatif selama proses loading/unloading model.

## Dokumen Terkait
- [AI_MODEL_LIFECYCLE.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/AI_MODEL_LIFECYCLE.md)
- [PROGRESSIVE_AI_PROCESSING.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/PROGRESSIVE_AI_PROCESSING.md)
- [DEVICE_CAPABILITY_MANAGER.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/architecture/DEVICE_CAPABILITY_MANAGER.md)
- [ADR-024-DEVICE-CAPABILITY-MANAGEMENT.md](file:///home/ahmadbadawi/Documents/CodinganKu/Android/KerjaiAja/docs/decisions/adr/ADR-024-DEVICE-CAPABILITY-MANAGEMENT.md)
