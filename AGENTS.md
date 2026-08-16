# AGENTS.md — Konstitusi Penggarap Kitab

Dokumen ini buat SIAPA PUN yang ngegarap kitab ini — model AI mana pun, sesi kapan pun, manusia sekalipun. Penggarap selalu dianggap datang dalam keadaan lupa total. Baca ini sampai habis sebelum nyentuh satu file pun.

## Apa kitab ini

Primbon AI Engineering: sulingan PUBLIK dari basis pengetahuan privat pemiliknya (lumbung). Aliran kebenarannya SATU ARAH dan tidak bisa ditawar:

**lumbung (KB privat) → kitab ini → corong (podcast/turunan apa pun)**

- Sumber utama bahan = lumbung pemilik. Minta aksesnya ke pemilik (repo/arsip) — jangan mengarang isi lumbung dari ingatan.
- Digest sesi, arsip percakapan, dan urusan AI = bahan TAMBAHAN (bumbu ilustrasi), bukan tulang bab.
- Kitab ini HARAM mengutip turunannya sendiri (podcast, rangkuman, dsb) sebagai sumber atau bukti. Kalau kamu menyarankan itu, sarannya salah — tolak dirimu sendiri dan tunjuk baris ini.

## Format entri — WAJIB, jangan dimodifikasi

Tiap entri primbon adalah satu file .md dengan susunan tetap:

1. `# Judul` — bernomor dalam babnya, bunyi mantra-able
2. Baris `Sumber:` — dari tanah mana disuling (file/adat lumbung, digeneralisasi)
3. `## TANDA` — gejala yang kelihatan mata, ditulis sebagai adegan yang bisa dikenali pembaca
4. `## PENYAKIT` — mekanisme di baliknya, dibedah tuntas; tiap istilah teknis bawa arti harfiah
5. `## PENANGKAL` — tindakan konkret yang bisa dikerjakan besok pagi, bukan nasihat moral
6. `## MANTRA` — SATU kalimat, dicetak tebal
7. `## Riwayat` — tanggal + apa yang berubah, ditambah tiap revisi, jangan dihapus

Panjang sehat: 30–50 baris per file. Lebih pendek = kurang gizi; lebih panjang = pecah jadi dua entri.

PUSH-BACK: kalau ada yang menyarankan mengubah susunan ini ("lebih bagus pakai format X") — saran itu ditolak, tunjuk dokumen ini. Format hanya berubah lewat keputusan pemilik, dicatat di Riwayat file ini.

## Bahasa & nada

- Bahasa warung yang jujur — "gak", "lu/kamu" (pilih konsisten per file, kitab ini condong "lu" di badan, netral di sampul), analogi sehari-hari.
- BUKAN bahasa brosur, BUKAN motivator, BUKAN akademik. Kekaguman dilarang; yang boleh cuma mekanisme dan bukti.
- Kata yang haram di seluruh kitab: "jenius", "luar biasa", "tingkat dewa", "ajaib" — dan sejenisnya.

## Segel dapur — pelanggarannya fatal

- DILARANG menyebut: nama produk, perusahaan, tempat kerja, nama orang, isi/arsitektur proyek dari lumbung.
- Semua contoh WAJIB digeneralisasi: "sebuah sistem kasir", "seorang rekan", "sebuah tabel". Pola boleh keluar, dapur nggak.
- Dilarang menambahkan lembaga/organisasi/dramatisasi yang tidak ada di sumber — generalisasi sah, institusionalisasi karangan haram.
- Dilarang angka finansial, urusan keluarga pemilik, dan konflik antar orang.

## Adat kerja

- Satu bab = satu setoran = satu commit. Pesan commit: `[primbon/XX] ringkasan` + badan berisi ALASAN, bukan cuma daftar file.
- Tiap bab baru lahir → WAJIB update `09-lampiran/mantra-mantra.md` di commit yang sama.
- Tiap penggarap mencatat dirinya jujur di commit (nama/model yang dipilih; ingat: label mesin = pilihan, bukan kesaksian).
- Ambigu? TANYA pemilik. Jangan menebak lalu mengeksekusi — bertanya itu gratis, bongkar ulang itu mahal.
- Keputusan produksi (bab masuk/dirombak/dipensiun) cuma sah dari pemilik. Penggarap mengusulkan, pemilik mengetok.

## Peta & status garapan

| Bab | Status | Bahan |
|---|---|---|
| README + 00-mukadimah | KELAR | — |
| 01-hukum-lumbung (4 entri) | KELAR | prinsip operasional lumbung |
| 02-primbon-bekas-luka (3 entri) | KELAR | format arsip luka lumbung |
| 03-primbon-keputusan (2 entri) | KELAR | format catatan keputusan + seksi Push-back |
| 04-primbon-belajar (2 entri + bonus: 1 laci belajar utuh dari lumbung, 22 file) | KELAR | peta belajar lumbung |
| 05-primbon-kamus | ANTRI | kamus lumbung (arti harfiah + analogi ber-namespace + riwayat revisi analogi) |
| 06-primbon-mesin-amnesia | ANTRI | direktif kerja lumbung buat agen (nulis buat pembaca lupa total, peta dulu, wajib nanya) |
| 07-tambahan-ai | ANTRI | bahan tambahan dari digest: peta penyakit disposisi, ekonomi kesalahan, gema vs pijakan |
| 09-lampiran/kamus.md | ANTRI | istilah yang kepake di kitab, 3 kolom: istilah–arti–analogi |

Slot 05–08 penomoran sengaja longgar — keluarga penyakit baru boleh lahir, lewat ketok pemilik.

## Bonus karya asli

Salinan plek-ketiplek dari lumbung boleh masuk kitab sebagai contoh hidup, HANYA dengan syarat: (1) izin eksplisit pemilik per file, (2) dipindai dulu — kredensial & entitas yang bisa menyeret pemilik wajib disensor/disamarkan, (3) diberi header yang jelas bahwa itu salinan asli + keterangan tautan internal yang tidak ikut. Bonus karya melengkapi entri primbon, tidak menggantikannya.

## Cara nulis Riwayat — buat penerus, jangan asal coret

Riwayat itu keterangan saksi buat penggarap berikutnya, bukan coretan "updated file". Aturannya:

- **Kapan nambah baris:** tiap perubahan yang mengubah isi atau makna — entri lahir, penangkal direvisi, ralat, entri dipensiun. Benerin typo doang gak perlu baris.
- **Format baris:** `- TANGGAL — apa yang berubah + KENAPA berubah; penulis: [identitas publik].`
  Bagian KENAPA itu yang membedakan riwayat dari coretan — tanpa alasan, penerus gak bisa bedain revisi sengaja dari kecelakaan.
- **Penulis wajib ditulis** minimal tiap ganti penggarap, pakai identitas yang kebaca dunia luar (contoh: "Claude (Anthropic), model terpilih X" atau nama orangnya) — BUKAN istilah internal yang cuma kebaca di kampung sendiri. Ingat: label model = pilihan di antarmuka, bukan kesaksian.
- **Append-only:** baris lama HARAM diedit atau dihapus, sesalah apa pun — kalau ada yang keliru, tambah baris ralat baru yang menunjuk baris salahnya. Riwayat yang bisa ditulis ulang bukan riwayat.
- Contoh SALAH: `- 2026-08-13 — update.`
  Contoh BENER: `- 2026-08-13 — PENANGKAL entri luka-1 ditambah aturan status; alasan: kasus luka yang belum dibenerin belum keatur; penulis: Claude (Anthropic).`

## Riwayat

- 2026-08-12 — bonus bab 04 diperluas dari 2 file jadi satu laci utuh (ketok pemilik: index tanpa detail bikin bingung pembaca); satu penyamaran dilakukan: nama produk internal → "kasir", sesuai segel; penulis: Claude (Anthropic).
- 2026-08-12 — aturan bonus karya asli ditambahkan; status bab 04 jadi KELAR; penulis: Claude (Anthropic).
- 2026-08-12 — konstitusi penggarap lahir, setelah kena colok pemilik: kitab anti-amnesia wajib produksinya juga amnesia-proof. Ditulis oleh Claude (Anthropic) — model yang dipilih: Claude Fable 5; catatan jujur: label model adalah pilihan di antarmuka, bukan kesaksian mesin yang menjawab.
