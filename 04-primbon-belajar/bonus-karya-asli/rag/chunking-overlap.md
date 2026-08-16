# Chunking Lanjutan — Overlap & Token-Based Split

> Lanjutan dari [rag-indexer.md](rag-indexer.md).
> Menangani section markdown yang melebihi batas ~500 token.

## Masalah yang Diselesaikan

Chunking by-heading bekerja baik, kecuali untuk section yang terlalu panjang (>500 token):

- Section panjang = 1 chunk raksasa
- Embedding-nya jadi "blur" — satu vector dipaksa menampung banyak makna sekaligus
- Akibatnya chunk itu tidak mirip dengan query manapun → jarang terangkat saat retrieval

Solusi: section yang melebihi batas dipotong lagi berdasarkan jumlah token.

## Masalah Baru: Potongan Buta

Memotong berdasarkan ukuran = memotong di posisi sembarang. Kalimat bisa terbelah:

```
teks: "... ID dibuat pakai hashlib. | Kalau pakai hash() bawaan, ID berubah tiap run ..."
                                    ↑ titik potong

chunk A: "... ID dibuat pakai hashlib."
chunk B: "Kalau pakai hash() bawaan, ID berubah tiap run ..."
```

Chunk B kehilangan konteks — "hash() bawaan" dibandingkan dengan apa?
Informasi yang seharusnya menyambung jadi terpisah di dua chunk.

## Solusi: Overlap

Ekor chunk sebelumnya diulang sebagai kepala chunk berikutnya:

```
tanpa overlap (max 50 token):        overlap 10 token:
chunk1: token 1–50                   chunk1: token 1–50
chunk2: token 51–100                 chunk2: token 41–90   ← mundur 10
chunk3: token 101–120                chunk3: token 81–120  ← mundur 10
```

- Kalimat yang terpotong di boundary dijamin utuh minimal di salah satu chunk
- Harganya: duplikasi data 10–20% → storage naik sedikit. Trade-off yang hampir selalu layak

## Contoh Mekanisme

Contoh generik — pakai kata sebagai pengganti token supaya mudah dilihat:

```python
def split_with_overlap(text, max_tok=6, overlap=2):
    words = text.split()
    chunks, start = [], 0
    while start < len(words):
        chunks.append(' '.join(words[start:start + max_tok]))
        start += max_tok - overlap   # maju 4, bukan 6
    return chunks
```

Isi variabel bila input `"a b c d e f g h i j"` (10 kata):

```
words  = ['a','b','c','d','e','f','g','h','i','j']
iterasi 1: start=0 → chunk 'a b c d e f'
iterasi 2: start=4 → chunk 'e f g h i j'   ← 'e f' terulang = overlap
hasil = ['a b c d e f', 'e f g h i j']
```

Catatan: di implementasi nyata, hitung pakai tokenizer (bukan `split()` per kata) —
jumlah kata ≠ jumlah token, tapi mekanismenya sama.

## Kenapa Overlap Jangan Terlalu Besar

Overlap 50% ≠ chunk jadi lebih blur — ukuran tiap chunk tetap sama, embedding per chunk tetap tajam.
Masalah sebenarnya: **duplikasi merusak hasil top-k saat retrieval.**

Chunk bertetangga isinya 50% sama → vector-nya mirip-mirip. Saat ada query yang
menyentuh area itu, mereka naik ranking bersama-sama:

```
query: "kenapa ID harus stabil"
top-3 hasil retrieval:
#1 chunk 12: "...ID pakai UUID... angka urut bentrok..."
#2 chunk 13: "...angka urut bentrok... merge aman..."     ← 50% = isi #1
#3 chunk 11: "...simpan ke db... ID pakai UUID..."        ← 50% = isi #1 juga
```

- 3 slot top-k terpakai, info uniknya hanya ~1.5 chunk
- Chunk lain yang mungkin relevan tergusur dari ranking
- Biaya tambahan: embedding call & storage naik ~2x saat indexing

Rumus mental: **overlap kecil = asuransi kalimat terpotong; overlap besar =
bayar 2x untuk mendengar info yang sama, sambil menggusur info lain dari top-k.**
Sweet spot: 10–20%.

### Dua penyakit yang sering tertukar

```
chunk KEGEDEAN   → embedding blur       (sisi indexing, per chunk)
overlap KEGEDEAN → top-k penuh duplikat (sisi query, antar chunk)
```

- Blur = penyakit chunk yang terlalu besar: 1 vector dipaksa menampung banyak makna
- Overlap TIDAK memperbesar chunk — ukuran tiap chunk tetap, embedding per chunk tetap tajam
- Yang rusak karena overlap besar bukan chunk-nya, tapi **komposisi hasil ranking**: slot rank 2–3
  yang seharusnya membawa makna baru terisi kalimat kembaran dari rank 1
- Analogi: bertanya ke 3 orang, tapi ketiganya membaca koran yang sama — semua jawaban jelas,
  tapi hanya dapat 1 sudut pandang, padahal bisa dapat 3

## Aturan Main untuk KB Markdown (Hybrid)

1. By-heading tetap jadi potongan utama — boundary semantik nomor satu
2. Hanya section yang melebihi batas (~500 token) yang dipotong lagi dengan overlap
3. Overlap TIDAK menyeberang antar section — beda heading = beda makna, tidak ada yang perlu disambung
4. Ukuran overlap: 10–20% dari ukuran chunk

## Riwayat

- 2026-07-18 — dibuat: konsep overlap + token-based split + aturan hybrid (diskusi R&D)
- 2026-07-20 — ditambah: section "Kenapa Overlap Jangan Terlalu Besar" + contoh top-k (diskusi R&D)
- 2026-07-20 — ditambah: sub-section "Dua penyakit yang sering tertukar" (blur vs duplikat top-k) + analogi koran
