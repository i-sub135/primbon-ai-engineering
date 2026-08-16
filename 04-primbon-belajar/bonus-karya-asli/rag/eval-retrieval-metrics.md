# Eval Formal Retrieval — recall@k & precision@k

> Keluarga **Eval formal 1/4 — konsep** di [MOL RAG](../mol-rag.md).
> Saudaranya: [ground truth](ground-truth.md) (2/4, selesai), implementasi (3/4) dan
> metrik sensitif urutan (4/4) — dua terakhir belum dikerjain.
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

**2. Precision buta urutan.**
Jawaban di peringkat 1 atau peringkat 10, `precision@k` tetap sama — isi keranjang tidak berubah,
hanya susunannya. Precision seperti menumpahkan keranjang ke meja lalu menghitung:
berapa persen yang guna. Urutan memang penting dan memang punya metrik sendiri,
tapi itu metrik ketiga (belum dipelajari), bukan precision.

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

## Analogi Patokan (punya Iyan)

```
recall@k    : di kampung Bojong Kenyot ada Samsul kagak
              — dari semua Samsul yang harusnya ketemu, berapa yang ketemu
              (3 Samsul, nemu 1 → 0.33)

precision@k : berapa Samsul dari total penduduk yang diangkut
              gak peduli dia tinggal di blok berapa gang mana
```

Patok penting: **kampung Bojong Kenyot itu keranjang hasil, bukan seluruh dunia.**

## Status & Sisa Utang

Keluarga eval formal ada 4 bagian:

**1/4 konsep — ketutup.** Isi dokumen ini.

**2/4 ground truth — ketutup.** → [ground-truth.md](ground-truth.md)

**3/4 implementasi — belum.** Yang masuk ke situ:

- Hitung `recall@k` + `precision@k` beneran pakai gold questions (kerjaan code)
- Cara memilih k yang wajar

**4/4 metrik sensitif urutan — belum.**
Recall & precision dua-duanya buta urutan — yang mengukur posisi jawaban di ranking
(MRR / NDCG) itu metrik lain.

## Riwayat

- 2026-08-06 — dibahas: recall@k, trade-off k vs noise, sambungan ke overlap (diskusi R&D)
- 2026-08-07 — dibuat: precision@k, jebakan (frekuensi, buta urutan, penyebut), analogi Bojong Kenyot (diskusi R&D)
- 2026-08-07 — eval formal dipecah jadi konsep (dokumen ini) + implementasi + metrik urutan, sebagai item terpisah di MOL
