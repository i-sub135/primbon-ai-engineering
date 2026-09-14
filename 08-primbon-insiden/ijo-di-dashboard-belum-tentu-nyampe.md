# Insiden 3 — Ijo di Dashboard Belum Tentu Nyampe

Sumber: kasus pertama di hub insiden lumbung asal (digeneralisasi) — layanan "1/1 healthy, nol restart, 24 jam uptime", padahal seluruh login produksi mati.

## TANDA

Semua indikator ijo: container hidup, health check lolos, nol restart, uptime sehari penuh. Sementara itu pengguna gak bisa login, tiap endpoint auth balikin error 500. Tim ngecek dashboard, bilang "sistem sehat", terus nyari salah di tempat lain — padahal yang salah justru definisi "sehat"-nya.

## PENYAKIT

Health check kebanyakan ngukur **proses hidup**, bukan **jalur kerja nyampe**: dia nanya "kamu masih napas?", bukan "kamu bisa nyampe database, cache, dan dunia luar?". Di kasus asalnya, kontainer sehat sempurna — yang rusak ada di lapisan jaringan di bawahnya: balasan dari luar udah dibalikin ke alamat kontainer, tapi tabel rute dikirim lagi ke luar, bukan ke dalam. Proses hidup, jalannya buntu, dan gak ada satu pun indikator yang ngukur jalannya. Dashboard ijo itu jujur — dia cuma ngejawab pertanyaan yang salah.

## PENANGKAL

- **Health = jalur ujung ke ujung**, bukan "kontainer up": cek yang beneran nyentuh database, cache, dan satu tujuan luar. Kalau salah satu putus, statusnya merah walau prosesnya hidup.
- **Bedain "hidup" dari "berfungsi"** di dashboard — dua lampu, bukan satu.
- **Pas gejala nyata gak cocok sama dashboard, yang dipercaya gejalanya.** Dashboard cuma ngejawab pertanyaan yang dulu diprogram buat dijawab.
- Sedia **cek cepat dua sisi** buat jalur keluar: kirim satu paket dari dalam kontainer sambil ngintip di antarmuka tuan rumah — nyampe apa nggak, baliknya lewat mana. Satu perintah, jawabannya pasti.

## MANTRA

**Ijo di dashboard belum tentu nyampe.**

## Riwayat
- 2026-09-14 — entri lahir; penulis: Claude (Anthropic).
