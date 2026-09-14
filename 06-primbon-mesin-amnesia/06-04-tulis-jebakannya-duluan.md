# Amnesia 4 — Ngerti Itu Kerjaan yang Baca, Tapi Jebakannya Tulis Duluan

Sumber: spesifikasi API yang ditulis kuli QA di lumbung asal buat rekan front-end — salinannya ada di `bonus-karya-asli/api-spec-feedback-chat.md`.

## TANDA

Spec dikirim, dibaca sekilas, dilempar mentah ke AI, hasilnya diintegrasikan. Seminggu kemudian: tiket bug "endpoint balikin 404" — padahal endpointnya emang belum ada, dan itu ditulis di paragraf kedua. Tiket bug berikutnya: "field feedback ada di pesan user, tombolnya nongol di tempat yang salah" — padahal itu juga ditulis, di bagian yang judulnya literally "tiga hal yang bikin FE salah kalau gak dibaca."

## PENYAKIT

Dokumen biasanya ditulis buat pembaca **kooperatif**: yang baca urut, yang inget status, yang nanya kalau bingung. Pembaca yang beneran ada itu kebalikannya — sibuk, males, dan sekarang punya AI buat nyari jalan pintas. Nulis buat pembaca kooperatif itu bukan sopan, itu **naif**: tiap jebakan yang penulis udah bisa tebak tapi dibiarin diem, bakal dibayar dua kali — sekali sama pembaca yang jatuh, sekali sama penulis yang ngurusin tiket bug palsunya. Dan yang paling mahal: kesalahan pembaca yang lempar-mentah-ke-AI itu **mendarat atas nama dia**, bukan atas nama modelnya — tapi jarang ada yang ngingetin sebelum kejadian.

## PENANGKAL

Tulis dokumen kayak orang yang udah tau pembacanya bakal males, dan bilang ke dia terang-terangan:

- **Status di kepala, huruf gede**: "BELUM ADA DI SERVER — nembak sekarang balasannya 404, dan itu bukan bug." Satu kalimat itu mematikan satu tiket bug sebelum lahir.
- **Jebakan yang udah ketebak, dinamain duluan** — bagian khusus berjudul jujur ("tiga hal yang bikin lu salah kalau gak dibaca"): field yang ada tapi harus disembunyiin, `null` yang punya dua arti, dua ID yang mirip tapi beda. Penulis udah tau di mana pembaca bakal jatuh; ngebiarin itu diem sama aja masang jebakan sendiri.
- **Yang diabaikan server disebut eksplisit** — "kirim field lain bakal diabaikan, bukan disimpan, dan ada testnya" — biar gak ada yang nyelundupin data lewat body terus heran kenapa ilang.
- **Tanggung jawab dinyatakan, bukan disiratkan**: *ngerti itu kerjaan lu, bukan kerjaan model yang lu suruh.* Sindiran boleh — asal tiap sindiran bawa mekanisme (status, tiket, alasan), bukan cuma nada.
- **Dokumen tetangga dijaga**: nyebut mana yang masih berlaku dan mana yang cuma belum lengkap, biar pembaca gak nganggep spec lama basi.

Ukurannya: dokumen yang bagus itu yang **nutup tiket bug sebelum tiketnya dibuka.** Kalau tiket yang masuk isinya hal yang udah ditulis, yang salah pembacanya; kalau isinya hal yang penulis udah tau tapi gak ditulis, yang salah dokumennya.

## MANTRA

**Ngerti itu kerjaan yang baca — tapi jebakannya tulis duluan.**

## Riwayat
- 2026-09-14 — entri lahir dari spec kuli QA pemilik; penulis: Claude (Anthropic).
