# Metrik Sensitif Urutan — MRR & NDCG

> Keluarga **Eval formal 4/4** di [MOL RAG](../mol-rag.md).
> Saudaranya: [konsep recall/precision + cara milih k](eval-retrieval-metrics.md) (1/4 & 3/4),
> [ground truth](ground-truth.md) (2/4).
> Menjawab: recall & precision dua-duanya buta urutan — pakai apa buat ngukur POSISI jawaban.
>
> **Status dokumen: SEPARUH.** MRR ketutup ulang 2026-09-01 (versi 08-29 dinyatakan keracunan
> oleh pemilik dan ditulis ulang total pakai jalur kelas yang beneran jalan). NDCG belum dibahas.

## Pintu Masuk — Kenapa Butuh Metrik Urutan

Frame **etalase warung mpok Ipeh** (punya pemilik). Catetan emak = [kopi, garem, gula, kecap].
Juned cuma bisa milih dari etalase.

**Senin** (k=10) — barang cocok di posisi 1, 2, 3
1. kopi ✓
2. gula ✓
3. kecap ✓
4. sabun
5. rokok
6. korek
7. mie
8. sampo
9. permen
10. teh

**Selasa** (k=10) — barang cocok di posisi 8, 9, 10
1. sabun
2. rokok
3. korek
4. mie
5. sampo
6. permen
7. teh
8. kopi ✓
9. gula ✓
10. kecap ✓

```
recall     Senin 3/4    Selasa 3/4     ← KEMBAR
precision  Senin 3/10   Selasa 3/10    ← KEMBAR
```

Isinya sama, posisinya beda, dua metrik lama gak bisa bedain. Kelihatannya bedanya
pas etalase **dipotong**. Rabu & Kamis, k=5, urutan sama kayak Senin & Selasa:

```
Rabu  (urutan Senin)   [ kopi ✓, gula ✓, kecap ✓, sabun, rokok ]   recall 3/4
Kamis (urutan Selasa)  [ sabun, rokok, korek, mie, sampo ]          recall 0/4
```

**Isi sama, posisi beda → etalase utuh keliatan kembar, etalase dipotong yang di belakang
mati.** Butuh angka yang bisa bedain Senin dan Selasa *sebelum* dipotong. Itu lubang yang
diisi metrik urutan.

## RR — Reciprocal Rank [kebalikan peringkat]

Bahan bakunya satu angka: **posisi barang cocok yang PERTAMA**. Senin = 1, Selasa = 8.
Syarat skornya: Senin > Selasa, dan posisi 1 = maksimal. Tiga percobaan di kelas:

```
percobaan 1   skor = posisi          Senin 1,  Selasa 8      ✗ kebalik
percobaan 2   skor = (k+1) − posisi  Senin 10, Selasa 3      ✓ lolos, tapi NEMPEL KE k
percobaan 3   skor = 1 / posisi      Senin 1.00, Selasa 0.125 ✓ lolos, BEBAS k
```

Percobaan 2 gugur pas etalase ganti panjang: k=10 pakai 11−posisi, k=20 pakai 21−posisi.
Skor Senin kemaren (10) gak bisa dibandingin sama Senin besok (20). Percobaan 3 gak peduli
k — posisi 1 selalu 1.00, posisi 8 selalu 0.125. Itu alasannya `1/posisi` yang dipakai
dunia: **satu-satunya cara balik yang gak ikut berubah pas etalase berubah.** Bukan angka sakti.

```
RR = 1 / posisi barang cocok PERTAMA di etalase

posisi 1   → 1.00
posisi 2   → 0.50
posisi 4   → 0.25
posisi 8   → 0.125
posisi 10  → 0.10
gak nongol → 0
```

**Kenapa gak-nongol = 0, bukan skor kecil.** Dua cara lihat:
1. Gak ada posisi → gak ada yang bisa dibalik. Rumusnya gak jalan, disepakati 0.
2. Posisi 100 → 0.01, posisi 1000 → 0.001 — makin belakang makin nyerempet 0 tapi gak pernah
   nol. "Gak nongol" harus lebih jelek dari posisi 1000 sekalipun → satu-satunya angka di
   bawah semua itu: 0. Sama persis kayak recall Kamis: Samsul gak ada → 0/4.

Catatan: "gak nongol" = gak nongol **di etalase sepanjang k**, bukan gak ada di gudang.
Barang di peringkat 47 dengan k=10 → RR 0. Jadi MRR sebenernya MRR@k, konsisten sama plateau.

Batasan RR: cuma lihat barang cocok **pertama**. Gula & kecap di belakangnya gak dihitung.
Itu yang nanti ditambal NDCG.

## MRR — Mean Reciprocal Rank [rata-rata kebalikan peringkat]

Pola dua level, sama kayak recall:

```
LEVEL 1 — per kedatangan Juned   RR = 1 / posisi
LEVEL 2 — laporan                MRR = rata-rata semua RR
```

Laporan mingguan mpok Ipeh:
```
Senin   posisi 1     → 1.00
Selasa  posisi 8     → 0.125
Jumat   posisi 4     → 0.25
Sabtu   gak nongol   → 0
                     ─────────
MRR = 1.375 / 4 = 0.34
```

## POV — Ini Rapor Buat Siapa (kunci analogi, punya pemilik)

pemilik mentok nyari padanannya sampai nemu sebabnya: **gak jelas berdiri sebagai siapa.**
Dua tokoh yang punya kepentingan beda:

```
POV EMAK      nagih belanjaan            "dapet apa kagak, bawa sampah berapa"
              → recall & precision        emak gak peduli kopi dipajang nomor 1 atau 8

POV PEMODAL   nuntut kerapihan etalase   "barang yang bener dipajang seberapa depan"
              → RR & MRR                  pemodal gak ngurusin Juned, ngurusin CARA NYUSUN
```

**MRR = rapor mpok Ipeh dari pemodal, bukan dari emak.** Yang dinilai cara nyusun etalase
(= retriever), bukan hasil belanja. pemilik gantian dua-duanya: ngecek hasil akhir sistem →
jadi emak; ngutak-atik cara nyusun (embedding, chunking, nanti rerank) → jadi pemodal.

Rapor pemilik buat MRR 0.34 (2026-09-01): *"kurang bagus — ada satu hari yang bener-bener gak
ketemu (Sabtu), asumsi catetan emak gak berubah."* Ditambah di kelas: Selasa (0.125) juga
nyeret — barang ADA tapi dipajang belakang. Tanpa Sabtu pun MRR cuma 1.375/3 = 0.46.
Dua penyakit beda: Sabtu = soal stok (wilayah emak/recall), Selasa = soal susunan (wilayah
pemodal). MRR gak bisa bedain keduanya, dia ngehukum dua-duanya sekaligus.

Versi pemilik (21:32), ini yang jadi cue:
```
Sabtu   → EMAK marah      : Juned balik tangan kosong        (stok)
Selasa  → INVESTOR ngamuk : barang laku dipajang di ujung    (susunan)
MRR     → kena PENTUNG DUA-DUANYA
```

## Asal-usul & Cara Baca di RAG

MRR lahir di *question answering* / search (TREC QA, 1999) — pembacanya **manusia**, yang
berhenti begitu nemu jawaban pertama. Makanya cuma yang PERTAMA dihitung dan kurvanya curam
di depan (1→2 turun 0.50, 9→10 turun 0.01).

Di RAG yang baca etalase itu **LLM**, dijejelin semua k potongan sekaligus. Posisi ngaruhnya
lewat jalur lain:
1. **Pemotongan hilir** — rerank / context builder sering cuma ambil top-n dari k → yang di
   belakang beneran kebuang. (Ini Rabu/Kamis.)
2. **Lost-in-the-middle** (belum dipelajari) — bentuk U: awal & AKHIR konteks kebaca, TENGAH
   yang buram. "Posisi 10 dari 10 gak kebaca" itu TIDAK tepat.

Jadi buat RAG, MRR paling jujur dibaca sebagai **nilai retriever-nya sendiri** — persis POV
pemodal di atas.

## Jebakan yang Sudah Ketemu

1. **Jangan ngarang posisi buat yang gak nongol.** RR 0 itu "gak ada di etalase", bukan
   "posisi paling belakang".
2. **Lompatan terbesar bukan geser urutan, tapi dari gak-ada jadi ada.** Posisi 4→2 nambah
   +0.25; gak nongol → posisi 1 nambah +1.00.
3. **Frame "juara/peringkat" gelap di pemilik** (2026-08-29). Pakai "pajangan ke berapa".
4. **Soal nyocokin list: yang cocok harus udah ditandain ✓** (teguran pemilik 2026-09-01) —
   yang diuji kalkulasinya, bukan pencocokan katanya.

## Sisa Utang

**NDCG — belum dibahas.** Yang perlu ditutup di situ:

- Kepanjangannya (*Normalized Discounted Cumulative Gain*) dan artinya per kata
- **Contoh motivasinya harus BARU.** Keranjang A vs B di atas udah kebedain MRR
  (RR 1.00 vs 0.11) — itu motivasi metrik urutan secara umum, bukan motivasi NDCG.
  Pasangan yang bikin NDCG perlu: jawaban PERTAMA di posisi sama, sisanya beda:

  ```
  C  [ GOLD, GOLD, x, x, x, x, x, x, x, x ]     RR = 1.00
  D  [ GOLD, x, x, x, x, x, x, x, x, GOLD ]     RR = 1.00   ← MRR buta di sini
  ```
- Bedanya sama MRR: MRR cuma lihat jawaban bener PERTAMA, NDCG ngitung semua yang bener
- *Gain* = relevansi boleh **bertingkat** (sangat cocok / lumayan / gak), bukan cuma bener-salah.
  Ini alasan kedua NDCG ada, dan belum kesebut di mana pun.
- Bobot yang mengecil makin ke bawah (bagian *Discounted*) — **kurvanya BUKAN 1/posisi**,
  tapi 1/log2(posisi+1): lebih landai dari RR. Jangan asumsi bentuknya sama.
- Kenapa perlu dinormalisasi (bagian *Normalized*)

## Riwayat

- 2026-08-29 — dibuat: MRR ketutup (RR per pertanyaan, kurva curam-di-depan + alasan desainnya, Mean sebagai level laporan, dua jebakan, frame etalase). NDCG masih kosong. Ditulis Kuli Dosen di bawah authority CRUD `12-LEARNING/` yang dikasih pemilik hari yang sama
- 2026-09-01 — review: (1) ditegasin MRR juga @k, "gak nongol" = gak nongol di etalase k; (2) alasan RAG buat kurva curam DIKOREKSI — versi lama bilang posisi belakang gak kebaca LLM, itu bentrok sama lost-in-the-middle (U-shape). Diganti: asal MRR dari QA/search (pembaca manusia), di RAG posisi ngaruh lewat pemotongan hilir + LITM; (3) sisa utang NDCG ditambah: butuh contoh motivasi baru (RR sama), gain bertingkat, discount log2 bukan 1/posisi
- 2026-09-01 — pemilik nyatain versi 08-29 keracunan → file DITULIS ULANG TOTAL pakai jalur kelas 09-01: Senin/Selasa (kembar) → Rabu/Kamis k=5 (yang belakang mati) → 3 percobaan rumus (1/posisi menang karena bebas k) → Sabtu=0 → MRR 0.34 → kunci analogi POV emak vs pemodal (punya pemilik). Ketutup 21:25, lolos cek rapor-pemodal
