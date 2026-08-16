# Belajar 2 — Naik Tangga Cuma Boleh Bawa Bukti Mentok

Sumber: alur keputusan penyetelan model di lumbung asal (layer 0 sampai 3: selalu mulai dari layer paling ringan).

## TANDA

Orang mau bikin AI-nya "lebih pinter", dan langkah pertamanya langsung yang paling berat: nyari tutorial fine-tune, ngitung sewa GPU, ngumpulin dataset — padahal dia belum pernah nyoba benerin cara nanyanya sendiri. Tiga minggu kemudian: dataset setengah jadi, GPU nganggur, dan masalah aslinya ternyata kelar diubah lewat satu paragraf instruksi.

## PENYAKIT

Eskalasi alat sebelum diagnosis. Solusi berat itu menggoda karena kedengeran serius — "kami melakukan fine-tuning" bunyinya lebih gagah daripada "kami benerin prompt-nya". Padahal makin berat layernya, makin mahal tiga-tiganya: ongkos bikin, ongkos salah, dan ongkos balikin (rollback). Loncat ke layer berat tanpa bukti layer ringan udah mentok itu bayar harga penuh buat masalah yang mungkin harganya gratis.

## PENANGKAL

Tangga penyetelan dari lumbung asal — wajib dinaiki urut, dilarang loncat:

- **Layer 0** — pakai bawaannya dulu, kenali perilakunya. Banyak "masalah model" ternyata cuma belum kenal.
- **Layer 1** — setelan konfigurasi: parameter perilaku (suhu jawaban, panjang konteks) dan instruksi sistem. Murah, bisa dibalikin dalam hitungan detik.
- **Layer 2** — rekayasa prompt: cara nyusun instruksi dan contoh. Masih gratis, dampaknya sering paling besar.
- **Layer 3** — fine-tune beneran. Baru boleh disentuh kalau layer 0–2 **terbukti mentok** — dan buktinya harus bisa ditunjuk, bukan "kayaknya kurang".

Aturan naiknya satu: tiap mau naik layer, bawa bukti mentoknya layer bawah. Gak ada bukti = belum boleh naik = balik turun nyoba lagi. Yang ketahan di tangga ini bukan kemajuan — yang ketahan cuma gengsi.

## MANTRA

**Naik tangga cuma boleh bawa bukti mentok.**

## Riwayat
- 2026-08-12 — entri lahir.
