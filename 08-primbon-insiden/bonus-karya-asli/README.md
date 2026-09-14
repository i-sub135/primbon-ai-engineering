# Bonus Karya Asli — Hub Insiden dari Lumbung

Folder ini salinan **laci insiden produksi** dari basis pengetahuan privat pemiliknya — README hub (aturan menulisnya) + kasus pertama: insiden jaringan yang bikin login produksi mati padahal semua indikator ijo. Disalin atas izin pemilik sebagai contoh hidup bab 08: catatan insiden yang isi utamanya belokan salah, bukan akar masalah.

**Yang diubah dari aslinya, demi segel:** nama perusahaan disamarkan ("kasir"); nama host & layanan diganti placeholder (`<worker-host>`, `<exit-node-host>`, `<db-host>`, `<backend-service>`); alamat IP publik & subnet diganti placeholder (`<public-ip-…>`, `<public-subnet>`); alamat tailnet diganti placeholder (`<tailnet-ip-…>`); nama penyedia payment gateway disamarkan; ID node diganti `<node-id>`. Alamat internal Docker (`172.18.x`) dan resolver publik (`1.1.1.1`) dibiarkan — bukan identitas. Log, urutan paket, stempel waktu, dan seluruh jebakan diagnostiknya utuh.

## Riwayat
- 2026-09-14 — salinan pertama, 2 file (hub + 1 kasus, per lumbung 12 Sep); pindaian sensor nol; penulis: Claude (Anthropic).
