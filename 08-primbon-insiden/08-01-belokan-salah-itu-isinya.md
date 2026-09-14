# Insiden 1 — Akar Masalah Gratis Setelah Ketemu; Belokan Salahnya yang Mahal

Sumber: hub insiden di lumbung asal — aturan menulisnya: jebakan diagnostik adalah isi utama, bukan lampiran.

## TANDA

Laporan insiden yang rapi: gejala, akar masalah, perbaikan, selesai. Enam jam debugging yang muter-muter — nyalahin DNS, nyalahin database, restart ini-itu — gak ada di kertas. Bulan depan kejadian mirip, tim jatuh di lubang yang sama, dengan urutan tebakan yang sama, dan baru nyampe ke akar setelah enam jam yang sama.

## PENYAKIT

Akar masalah itu barang yang **kelihatan jelas setelah ketemu** — makanya dia yang paling gampang ditulis dan paling murah nilainya. Yang mahal beneran adalah **tebakan yang kelihatan meyakinkan tapi salah**: tiap belokan itu dibayar pake jam kerja, dan justru itu yang ingatan buang duluan, soalnya malu dan soalnya "udah gak relevan". Laporan yang cuma nyimpen akar masalah itu nyimpen bagian termurah dari insiden dan ngebuang bagian termahalnya.

## PENANGKAL

Catatan insiden pake kerangka tetap enam bagian, dan bagian ketiga yang paling gendut:

1. **Ringkasan** — gejala, dampak, rentang waktu.
2. **Mekanisme** — gimana rusaknya, pake bukti (potongan log, angka asli, tangkapan paket).
3. **Jebakan diagnostik** — SETIAP tebakan yang salah: kenapa dia kelihatan meyakinkan, dan realitanya apa. Termasuk yang memalukan. Ini isi utama, bukan lampiran.
4. **Cek cepat & obat** — perintah konkret yang bisa dijalanin pas kejadian lagi.
5. **Yang masih terbuka** — apa yang belum beneran diperbaiki.
6. **Pelajaran umum** — pola yang berlaku di luar kasus ini.

Ukurannya gampang: kalau seseorang yang gak ikut kejadian bisa **ngelewatin semua belokan salah** cuma dengan baca catatannya, catatannya bener. Kalau dia cuma tau akarnya tapi tetep bakal muter dulu, itu baru setengah catatan.

## MANTRA

**Akar masalah gratis setelah ketemu; belokan salahnya yang mahal.**

## Riwayat
- 2026-09-14 — entri lahir, mengisi kursi kosong bab 08; penulis: Claude (Anthropic).
