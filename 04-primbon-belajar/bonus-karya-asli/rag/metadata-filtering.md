# Metadata Filtering — Membatasi Pencarian per Source/Folder

> Lanjutan dari [rag-key-takeaways.md](rag-key-takeaways.md) (retrieval & top-k).
> Menggunakan metadata chunk sebagai saringan saat query, bukan hanya untuk citation.

## Poin 1 — Masalah: Query Mencari ke SEMUA Chunk

Tanpa filter, setiap query membandingkan vector-nya ke seluruh isi collection,
tidak peduli asal filenya:

```
KB berisi:
- catatan/masak/    → 40 chunk
- catatan/keuangan/ → 30 chunk
- catatan/olahraga/ → 30 chunk

query: "cara ngatur porsi mingguan"
       ↓ dibandingkan ke 100 chunk sekaligus
top-3 bisa tercampur: porsi MAKAN (masak),
porsi ANGGARAN (keuangan), porsi LATIHAN (olahraga)
```

Kata "porsi" maknanya dekat di ketiga topik → ranking bisa mengangkat chunk dari
folder yang salah, padahal user tahu jawabannya seharusnya dari `keuangan/`.

## Poin 2 — Setiap Chunk Membawa "KTP" (Metadata)

Saat indexing, chunk tidak disimpan telanjang — ia tersimpan bersama metadata:

```
id:        "keu-042"
embedding: [0.12, -0.83, ...]        ← vector untuk similarity
document:  "Anggaran mingguan dibagi per pos..."
metadata:  {"source": "catatan/keuangan/anggaran.md"}   ← KTP-nya
```

Poin kunci:

- Metadata menempel **dari lahir** — dicatat saat indexing, bukan saat query
- Dua fungsi metadata: **display** (menampilkan asal kutipan/citation) dan
  **filter** (menyaring pencarian). Barangnya sama, fungsinya beda
- Query hanya bisa MEMBACA metadata yang sudah ada — tidak bisa mengarang.
  Metadata yang lupa disimpan saat indexing = filter yang tidak akan pernah bisa;
  satu-satunya jalan: re-index
- Karena itu "metadata apa saja yang disimpan" adalah **keputusan desain indexing**,
  bukan urusan belakangan

## Poin 3 — `where`: Filter Berjalan Duluan, Ranking Belakangan

`col.query` = ranking, bukan filter — similarity tidak pernah menolak chunk,
hanya mengurutkan. Filter sesungguhnya masuk lewat parameter `where`:

```python
hasil = col.query(
    query_texts=["cara mengatur anggaran"],
    n_results=3,
    where={"source": "catatan/keuangan/anggaran.md"}   # ← saringan KTP
)
```

Urutan kejadian di dalam:

```
100 chunk di collection
   ↓ where: cek KTP dulu
12 chunk lolos (yang source-nya cocok)
   ↓ similarity ranking
top-3 dipilih HANYA dari 12 itu
```

Dua alat ini bekerja di lapis berbeda:

```
where      → saringan KERAS: KTP tidak cocok = gugur, semirip apapun isinya
similarity → urutan LUNAK: yang lolos saringan diranking dari paling mirip
```

Analogi: cek dulu alamat si bocah, baru tanya siapa bocahnya.
Dua keuntungan filter:

1. Scope pencarian mengecil — lebih cepat, tanpa noise lintas kampung
2. Nama ambigu (mis. "samsul") hanya ter-match di kampung itu saja —
   chunk dari kampung lain mustahil muncul, semirip apapun isinya

## Poin 4 — Filter per Folder: Field Metadata Didesain dari Lahir

`where={"source": "...anggaran.md"}` adalah match PERSIS — satu file saja.
Vector store itu **pedant literal**: ia hanya mencocokkan nilai metadata utuh.
Ia TIDAK bisa berpikir "alamat `Jl. Mawar 5, Desa Cikuya, Kec. Bojong Kenyot`
mengandung Bojong Kenyot" — baginya nilai panjang itu satu bongkahan: cocok atau tidak.

Kalau butuh "cari dari SEMUA file di `keuangan/`" tapi KTP hanya menyimpan path
lengkap, pilihannya:

```
Opsi kerja rodi:  enumerasi manual tiap kemungkinan
                  where={"source": {"$in": ["desa-cikuya.md", "desa-anu.md", ...]}}
                  → jalan, TAPI harus tahu daftar lengkapnya + melelahkan

Opsi KTP benar:   dari lahir KTP diberi field terpisah
                  where={"kecamatan": "bojong-kenyot"}
                  → satu pertanyaan, selesai
```

Desain metadata saat indexing:

```
metadata: {
  "source": "catatan/keuangan/anggaran.md",   ← untuk citation, match 1 file
  "folder": "keuangan"                        ← untuk filter per kampung/folder
}
```

Contoh query:

```python
where={"folder": "keuangan"}                        # semua file di keuangan/
where={"folder": {"$in": ["keuangan", "masak"]}}    # 2 folder sekaligus
```

Rumus: **1 kebutuhan filter = 1 field metadata.** Mau bisa filter per tahun?
Simpan `"year": 2024` saat indexing. Field yang tidak pernah disimpan =
filter yang tidak akan pernah bisa.

## Poin 5 — Aturan Main: Filter Itu Pisau, Bisa Salah Potong

Bahayanya satu: **filter salah = jawaban benar ikut terbuang, DIAM-DIAM.**

```
query: "berapa biaya bulanan gym"
asumsi: "ini soal duit" → where={"folder": "keuangan"}

padahal catatannya ada di: catatan/olahraga/membership-gym.md
                                    ↑ terbuang di gerbang KTP

hasil: top-3 terisi chunk keuangan yang menyerempet saja.
Tidak ada error, tidak ada warning — jawaban asli tidak pernah ikut lomba.
```

Kenapa kesalahan filter lebih susah ketahuan dibanding kesalahan ranking:

```
ranking salah → jawaban benar masih ADA di daftar (rank 5, rank 7);
                naikkan top-k / scroll → ketemu, ketahuan ada yang aneh
filter salah  → jawaban benar HILANG TANPA JEJAK;
                top-k dinaikkan jadi 100 pun tetap tidak muncul —
                sudah gugur di gerbang (filter = equals, kaku)
```

Yang membuat invisible: tidak ada error, tidak ada hasil kosong, top-k tetap
terisi chunk yang tampak meyakinkan. Output terlihat NORMAL padahal jawaban
aslinya tidak pernah ikut bertanding.

Aturan main:

1. Filter hanya dipakai kalau scope-nya **PASTI** — user menyebut eksplisit
   ("cari di catatan keuangan"), atau konteks aplikasi menjamin
   (tab "Keuangan" sedang terbuka)
2. Pertanyaan umum / eksplorasi → **jangan** filter, biarkan similarity bekerja
   di seluruh collection
3. Hasil filter kosong / jelek → fallback: ulangi query TANPA filter,
   jangan menyerah di saringan

Rumus mental: **filter = janji ke sistem "jawabannya PASTI di sini".
Berani janji hanya kalau benar-benar tahu.**

## Riwayat

- 2026-07-20 — dibuat: 5 poin metadata filtering (masalah → KTP → where → desain field → aturan main) + analogi KTP/kampung (diskusi R&D)
