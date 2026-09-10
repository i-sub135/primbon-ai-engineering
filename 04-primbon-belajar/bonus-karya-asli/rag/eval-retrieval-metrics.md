# Eval Formal Retrieval — recall@k & precision@k

> Keluarga **Eval formal 1/4 — konsep** di [MOL RAG](../mol-rag.md).
> Saudaranya: [ground truth](ground-truth.md) (2/4, selesai), implementasi (3/4) dan
> [metrik sensitif urutan](eval-ranking-metrics.md) (4/4, separuh: MRR sudah, NDCG belum).
> Bagian **"Cara Milih k"** (separuh dari 3/4) ikut nebeng di dokumen ini — rumah
> konsepnya memang di sini. Separuh sisanya (nulis code penghitung) belum dikerjain.
> Lanjutan dari testing 2 lapisan di [rag-key-takeaways.md](rag-key-takeaways.md) — bagian gold questions.
> Menjawab: gimana tau retrieval bagus atau nggak pakai angka, bukan feeling.

## Masalah yang Diselesaikan

Testing gold questions masih manual — lihat hasil retrieval, rasakan "kayaknya udah bener".
Tidak bisa dipakai untuk membandingkan dua konfigurasi (chunk size, overlap, model embedding).
Butuh angka supaya perubahan bisa diukur: naik atau turun.

## Dua Angka, Satu Pembilang

```
pembilang (SAMA untuk keduanya) : cacah potongan guna yang kebawa

recall@k    = pembilang / SEMUA potongan yang seharusnya kebawa
precision@k = pembilang / k  (semua yang diangkut)
```

Bedanya hanya penyebut. Itu satu-satunya perbedaan.

Beda pertanyaan yang dijawab:

```
recall@k    : "jawabannya kebawa gak?"       → k digedein: NAIK atau tetap
precision@k : "yang kebawa isinya guna gak?" → k digedein: TURUN
```

## Kenapa Harus Berdua

`recall@k` bisa dibohongi. Angkut semua chunk di database, recall pasti 1.00 —
sempurna, padahal tidak menyaring apa-apa. Recall tidak pernah turun kalau k digedein,
hanya naik atau tetap. Jadi angka bagus tidak menjamin retrieval bagus.

`precision@k` bergerak ke arah sebaliknya, jadi keduanya saling mengerem:
"iya kebawa sih, tapi mengangkut 9 sampah untuk mendapat 1 barang."

## Keypoint: Satu Tuas (k), Dua Arah Berlawanan

```
k DIGEDEIN  (truk digedein)  : recall seneng (naik/tetap) — precision nangis (turun)
k DIKECILIN (truk dikecilin) : precision seneng           — recall terancam
```

- Recall **tidak pernah turun** kalau k naik — gedein truk itu senjatanya recall:
  Samsul yang tadinya ketinggalan di pinggir jalan dapat peluang keangkut.
- Precision arah umumnya turun kalau k naik: pembagi (isi truk) pasti nambah,
  tambahan Samsul paling banter sisa yang belum keangkut.
- Konsekuensi: **k saja tidak bisa menyenangkan dua-duanya.** Milih k bukan cari
  angka cantik — cara milihnya dibahas di bagian implementasi (Eval formal 3/4).

## Contoh Mekanisme

```
Pertanyaan     : "kenapa harus makan teratur?"
Jawaban bener  : "makan yang teratur membuat sehat"   (1 potongan)
```

Cara A — ambil 3:

```
1. sarapan sebaiknya jam 7
2. makan malam jangan kekenyangan
3. makan yang teratur membuat sehat   ← jawabannya

guna : 1        sampah : 2
recall@3    = 1/1 = 1.00
precision@3 = 1/3 = 0.33
```

Cara B — ambil 10 (isi 1–3 sama, ditambah 7 kalimat tidak relevan):

```
guna : 1        sampah : 9
recall@10    = 1/1  = 1.00   ← sama seperti Cara A
precision@10 = 1/10 = 0.10   ← turun
```

Recall tidak bisa membedakan A dan B. Precision langsung membedakan.

## Kalau Jawaban Bener Lebih dari Satu Potongan

Di sini recall berhenti jadi 0/1:

```
Jawaban bener : "makan yang teratur membuat sehat"
                "telat makan bikin lambung perih"     (2 potongan)

Ambil 5, dua-duanya kebawa:
recall@5    = 2/2 = 1.00
precision@5 = 2/5 = 0.40

Ambil 5, cuma satu yang kebawa:
recall@5    = 1/2 = 0.50     ← muncul setengah
precision@5 = 1/5 = 0.20
```

Konsekuensinya: `precision = recall / k` **bukan rumus** — itu hanya kebetulan benar
ketika jawaban benarnya persis satu potongan. Pembilang sebenarnya adalah cacah
potongan guna, bukan nilai recall.

## Jebakan yang Sudah Ketemu

**1. Recall bukan "seberapa sering muncul di k".**
Per satu pertanyaan yang diukur hanya ada/tidak (kalau jawaban benarnya satu potongan).
Nongol sekali atau tiga kali nilainya sama. "Seberapa sering"-nya baru muncul di lapis atas:
dari 20 gold question, berapa yang jawabannya kebawa. Sering-nya lintas pertanyaan,
bukan di dalam satu hasil.

**2. Precision buta urutan — dan recall juga.**
Jawaban di peringkat 1 atau peringkat 10, `precision@k` tetap sama — isi keranjang tidak berubah,
hanya susunannya. Precision seperti menumpahkan keranjang ke meja lalu menghitung:
berapa persen yang guna. Urutan memang penting dan memang punya metrik sendiri —
MRR (sudah, lihat [eval-ranking-metrics.md](eval-ranking-metrics.md)) dan NDCG (belum) — bukan precision.

Jangan salah kira recall lebih peka: dua-duanya sama butanya.

```
keranjang A: [ GOLD, GOLD, x, x, x, x, x, x, x, x ]   ← jawaban di slot 1-2
keranjang B: [ x, x, x, x, x, x, x, x, GOLD, GOLD ]   ← jawaban di slot 9-10

recall    A = B        precision A = B        ← identik. dua metrik ini gak bisa bedain.
```

Padahal A jelas lebih bagus. Yang bisa membedakan A dan B baru muncul di MRR / NDCG.

**3. Penyebut precision = keranjang hasil (k), bukan seluruh database.**
Kalau mengangkut 10 chunk, penyebutnya 10 — bukan jumlah total chunk yang diindeks.

## Kenapa Sampah di Top-k Itu Masalah Nyata

Bukan cuma jelek di atas kertas. Semua k potongan itu dijejalkan ke model untuk menyusun jawaban:

- Model harus memilih sendiri mana yang relevan dari tumpukan k
- Makin banyak sampah, makin mudah model menyomot yang salah
- Ditambah lebih lambat dan lebih mahal (token context membengkak)

Analogi: mencari obat di kotak P3K isi 3 vs lemari isi 50. Obatnya ada di keduanya,
tapi di lemari isi 50 lebih mudah salah ambil.

## Sambungan ke Chunking Lanjutan (Overlap)

`overlap-chunk` yang terlalu besar membuat jatah k terpakai isi kembar —
ambil 10, yang benar-benar beda isinya mungkin cuma 6.
Efeknya terbaca langsung di `precision@k`: slot terpakai, informasi unik tidak nambah.
Lihat [chunking-overlap.md](chunking-overlap.md) → "Dua penyakit yang sering tertukar".

Jadi dua metrik ini juga berfungsi sebagai alat ukur untuk keputusan chunking/overlap,
bukan cuma laporan akhir.

## Cara Milih k

> Diulang dari nol 2026-09-01 (versi 08-29 dinyatakan keracunan oleh pemilik — nyender ke
> rerank/filter yang belum diajarin, contoh ambigu). Urutan di bawah = urutan yang
> beneran jalan di kelas, pakai contoh buatan pemilik sendiri.

Trade-off di atas cuma bilang "k gak bisa nyenengin dua-duanya". Bagian ini jawab:
angkanya berapa, dan gimana tau dari laporan doang.

### 1. Satu nama gak nongol — dua sebab, dua obat

Contoh pemilik: GT = [Samsul, Udin, Tarno], keranjang k=20 isinya cuma [Samsul, Udin].

```
sebab #1  k kurang lebar        Tarno ada di kampung, kepotong di luar 20
                                → obat: lebarin k
sebab #2  Tarno gak di kampung  a. penggaris salah — GT nulis Tarno, dia di kampung lain
                                b. kampungnya bolong — datanya gak pernah masuk
                                   (gak di-ingest, kepotong chunking)
                                → lebarin k = buang tenaga, mau k=1 juta juga gak nemu
```

### 2. Bedain #1 dan #2 tanpa turun ke lapangan: laporan beberapa k

Analogi pemilik: **tukang sensus yang gak turun lapangan** — modalnya cuma laporan.
Jadi minta laporan bukan satu k, tapi beberapa:

```
k=5    → recall 0.33   (Samsul)
k=10   → recall 0.67   (Samsul, Udin)   ← kenaikan terakhir
k=20   → recall 0.67
k=50   → recall 0.67
k=100  → recall 0.67                     ← dilebarin 10x, nol
```

- Angka masih naik → masih ada yang kepotong (sebab #1), lebarin lagi.
- Angka diem walau k dilebarin berkali-kali → **plateau**. Sisanya kena sebab #2,
  obatnya di hulu (chunking, embedding, ground truth), bukan di k.

### 3. Aturan: k terkecil yang udah nyentuh plateau = titik siku

Di laporan atas, kenaikan terakhir 5 → 10. Di 10 kurva nekuk → **titik siku** (knee point).
k produksi = 10.

### 4. Bantalan di atas siku boleh, tapi ada harganya

pemilik milih 20 sebagai bantalan ("kalau-kalau"). Sah — kurva asli gak semulus contoh —
asal tau bayarnya:

```
k=10 → k=20, recall tetep 0.67

bayar 1  precision 2/10 → 2/20 = anjlok setengah
         (10 slot tambahan keisi orang lain, bukan kosong)
bayar 2  beban angkut: 20 orang harus dibawa & diperiksa yang baca keranjang
         → token, waktu, biaya naik 2x buat recall yang sama
```

Kelewat 50–100 jelas rugi: bayar 5–10x, dapet nol.

### 5. k itu garis potong, bukan kualitas mesin

Menaikkan k **tidak membuat retrieval jadi lebih pintar** — rankingnya sama persis,
yang berubah hanya sampai mana daftar itu dipotong.

```
k=5    Q: [ a  b  c  d  e ]                          ❌ patokan gak kebawa
k=10   Q: [ a  b  c  d  e  f  g  GOLD  h  i ]        ✅ kebawa

GOLD tetap di peringkat 8 di dua-duanya. Yang beda cuma panjang potongannya.
```

### 6. Beda level itungan (jebakan yang bikin pusing)

```
LEVEL 1 — per satu pertanyaan
  patokan 1 nama   → recall 1 atau 0 (binary)
  patokan 3 nama   → recall 2/3 dst — tetep "ada kagak", dihitung per nama
  precision        → 0/k, 1/k, ...  gak pernah binary, pembaginya k

LEVEL 2 — laporan
  recall  = rata-rata dari semua kartu level 1
```

### Belum diajarin: kenapa recall dimenangin duluan (asimetri kerugian)

Ditunda 2026-09-01. Alasannya butuh tahap **sortir sesudah retrieval** (rerank/filter)
yang belum masuk MOL — tanpa itu penjelasannya lompat. Analogi jaring tebar disimpen
di bawah buat nanti, jangan dipakai sebagai cue dulu.

## Analogi Patokan (punya pemilik)

```
recall@k    : di kampung Bojong Kenyot ada Samsul kagak
              — dari semua Samsul yang harusnya ketemu, berapa yang ketemu
              (3 Samsul, nemu 1 → 0.33)

precision@k : berapa Samsul dari total penduduk yang diangkut
              gak peduli dia tinggal di blok berapa gang mana
```

Patok penting: **kampung Bojong Kenyot itu keranjang hasil, bukan seluruh dunia.**

### Juned & Warung Mpok Ipeh — analogi untuk k dan plateau

```
catetan emak    = [kopi, garem, gula, kecap]   → ground truth (4 patokan)
gudang mpok Ipeh = seluruh stok warung          → index. TETAP, bukan k
etalase         = barang yang disodorin ke Juned → k (di contoh ini 10 barang)
                  disusun ulang tiap Juned dateng, sesuai catetan yang dibawa
                  yang paling mirip ditaro paling depan
yang dapet      = [kopi, gula]                  → 2

recall    = 2/4  = 0.50   ← pembagi: catetan emak
precision = 2/10 = 0.20   ← pembagi: isi etalase
```

Dua patok yang gampang ketuker:

- **k = panjang etalase, bukan kapasitas gudang.** Gudang boleh isi 10.000, k tetap 10.
- **Plateau = "mpok Ipeh emang gak nyetok garem".** Mau etalase dipanjangin sampai
  bongkar semua rak, garem tetap gak nongol. Obatnya nyetok ulang warung, bukan
  manjangin etalase.

### Jaring Tebar — analogi untuk asimetri (DISIMPEN, belum diajarin — lihat "Belum diajarin" di atas)

```
sampah kejaring, udah naik ke perahu → masih bisa disortir
ikan target lolos dari lubang jaring → kabur ke laut, gak bisa diapa-apain
                                       kecuali ganti jaring & tebar ulang
```

Itu alasan recall dimenangkan duluan.

## Status & Sisa Utang

Keluarga eval formal ada 4 bagian:

**1/4 konsep — ketutup.** Isi dokumen ini.

**2/4 ground truth — ketutup.** → [ground-truth.md](ground-truth.md)

**3/4 implementasi — separuh.**

- ✅ Cara memilih k yang wajar → bagian "Cara Milih k" di dokumen ini (diulang & ketutup 2026-09-01; asimetri kerugian ditunda sampai rerank)
- ⬜ Hitung `recall@k` + `precision@k` beneran pakai gold questions (kerjaan code)

**4/4 metrik sensitif urutan — separuh.** → [eval-ranking-metrics.md](eval-ranking-metrics.md)

- ✅ MRR
- ⬜ NDCG

## Riwayat

- 2026-08-06 — dibahas: recall@k, trade-off k vs noise, sambungan ke overlap (diskusi R&D)
- 2026-08-07 — dibuat: precision@k, jebakan (frekuensi, buta urutan, penyebut), analogi Bojong Kenyot (diskusi R&D)
- 2026-08-07 — eval formal dipecah jadi konsep (dokumen ini) + implementasi + metrik urutan, sebagai item terpisah di MOL
- 2026-08-17 — ditambah keypoint trade-off k: satu tuas dua arah berlawanan (recall naik/tetap vs precision turun), permintaan pemilik saat sesi cara milih k
- 2026-08-29 — ditambah bagian "Cara Milih k" (asimetri kerugian, k = garis potong, aturan titik siku, plateau bukan penyakit k, beda level itungan binary vs rata-rata); jebakan buta-urutan diperluas: recall juga buta urutan, bukan cuma precision; analogi baru punya pemilik masuk — Juned & warung mpok Ipeh (k = etalase, gudang = index, plateau = barang gak distok) dan jaring tebar (asimetri). Izin tulis dari pemilik di sesi yang sama
- 2026-09-01 — review: pointer ke metrik urutan dibenerin (status 4/4 dari "belum" jadi separuh + link ke eval-ranking-metrics.md; jebakan buta-urutan nunjuk MRR yang udah dipelajari); ukuran etalase (10) ditulis eksplisit di blok Juned
- 2026-09-01 — "Cara Milih k" ditulis ulang dari nol setelah rollback: urutan baru (dua sebab → laporan multi-k → plateau → titik siku → bantalan & harganya), contoh Samsul/Udin/Tarno + analogi "tukang sensus gak turun lapangan" punya pemilik. Asimetri kerugian dipisah jadi "belum diajarin" (nunggu rerank). Ketutup 19:52, lolos cek-by-analogi tanpa dituntun
