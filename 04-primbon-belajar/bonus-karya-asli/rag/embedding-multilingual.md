# Embedding Multilingual — Saat Model Punya "Bahasa Ibu"

> Lanjutan dari [metadata-filtering.md](metadata-filtering.md).
> Menguji dan menangani gap kualitas retrieval antara query/chunk bahasa Indonesia
> vs bahasa Inggris pada model embedding English-heavy (mis. `nomic-embed-text`).

## Poin 1 — Model Embedding Punya "Bahasa Ibu"

Model embedding juga model — ia belajar dari data training. `nomic-embed-text`
trainingnya mayoritas teks Inggris. Konsekuensinya seperti juru arsip yang jago
bahasa Inggris tapi bahasa Indonesianya pas-pasan:

```
kalimat EN masuk → makna tertangkap AKURAT      → vector tajam
kalimat ID masuk → makna tertangkap "kira-kira" → vector buram
```

Seluruh sistem retrieval berdiri di atas satu asumsi: **makna mirip = vector dekat**.
Kalau juru arsipnya tidak benar-benar paham bahasanya, asumsi itu goyah:

```
query:    "cara mengatur anggaran bulanan"      (ID)
chunk 1:  "Anggaran bulanan dibagi per pos..."  (ID)
chunk 2:  "Set a monthly budget per category"   (EN)

harapan:  chunk 1 & 2 dua-duanya dekat ke query (maknanya sama persis)
risiko:   model English-heavy mengukur jarak ID meleset —
          chunk relevan dapat skor biasa saja, kalah oleh chunk lain
          yang kebetulan "terdengar mirip" di telinga si model
```

**Bukti lapangan (kasus nyata, 2026-07-20):** mesin `memory_search` OpenClaw pakai
nomic. MEMORY.md yang ditulis bahasa Inggris → recall agent bagus; ditulis bahasa
Indonesia → jarang ter-recall, *padahal isinya lebih tajam*. Gejala persis
"juru arsip berbahasa ibu Inggris".

Relevan untuk KB campur ID/EN (istilah teknis EN, penjelasan ID, query biasanya ID).

## Poin 2 — Diagnosis: Tes A/B Query Kembar, 1 Makna 2 Bahasa

Jangan percaya feeling — buktikan. Prinsipnya sama dengan gold questions
(rag-key-takeaways), ditambah 1 variabel: bahasa.

```
1. Pilih chunk yang SEHARUSNYA jadi jawaban (gold chunk):
   "Anggaran bulanan dibagi per pos: makan, transport, darurat..."

2. Buat 2 query yang maknanya SAMA PERSIS:
   query EN: "how to split monthly budget"
   query ID: "cara membagi anggaran bulanan"

3. Jalankan keduanya, catat posisi si gold chunk:
   query EN → gold chunk muncul di rank 1, distance 0.31
   query ID → gold chunk muncul di rank 4, distance 0.55   ← gejala!
```

Cara baca: makna query sama, chunk sama — yang beda HANYA bahasa. Kalau rank
anjlok saat ID, tersangka tinggal satu: model memang lemah bahasa Indonesia.
Ulangi untuk 5–10 gold chunk; pola konsisten (ID selalu lebih jeblok) = diagnosis terkunci.

### Kenapa makna kedua query harus sama persis (variabel terkontrol)

Analogi timbangan: mau menguji "timbangan ini akurat tidak?"

```
Cara benar:  beras 1 kg yang SAMA ditaruh di timbangan A, lalu timbangan B.
             Hasil beda? → yang salah pasti TIMBANGANNYA.
Cara salah:  beras di timbangan A, gula di timbangan B.
             Hasil beda? → timbangan salah ATAU barangnya memang beda —
             dua tersangka, tidak bisa menuduh. Tesnya sia-sia.
```

Rumus: **untuk menuduh satu hal, semua hal lain harus dibuat sama —
supaya tersangkanya tinggal satu.** Berlaku untuk semua eksperimen, bukan cuma RAG.

Analogi masa jenis: makna = masa jenis, bahasa = bentuk besinya. Bentuk boleh
beda, masa jenis harus terbaca sama — kalau alat ukur bilang beda, yang
bermasalah alat ukurnya. Itulah definisi model multilingual yang bagus:
membaca makna, cuek terhadap bahasa.

### Jebakan angka: rank vs distance

- Distance hanya bisa dibandingkan DI DALAM model yang sama.
  0.55 versi nomic vs 0.40 versi model lain = tidak nyambung, beda penggaris.
- Metrik yang jujur untuk dibandingkan lintas kondisi: **rank**
  (posisi gold chunk), bukan angka distance.

## Poin 3 — Obat: Dua Jalur, Beda Harga

```
Jalur 1 — GANTI JURU ARSIP (model multilingual, mis. bge-m3)
  + makna ID & EN terbaca setara — akar masalah sembuh
  - model lebih besar & lambat dari nomic
  - WAJIB re-index TOTAL (lihat bagian di bawah)

Jalur 2 — AKALI TANPA GANTI MODEL
  a. tulis catatan condong EN (trik MEMORY.md yang terbukti jalan)
  b. translate query ke EN dulu sebelum di-embed
  + murah, tanpa re-index
  - menambah 1 langkah & 1 titik gagal (translate bisa melenceng),
    dan sifatnya "mengalah" pada keterbatasan model, bukan menyembuhkan
```

Urutan wajib: **diagnosis dulu (tes A/B) → baru pilih obat.** Kalau gap-nya
kecil, jangan ganti apa-apa — ganti model = bayar re-index total; jangan
dibayar untuk penyakit yang tidak terbukti.

### Kenapa ganti model = re-index total (vector lama & baru tidak boleh campur)

Vector bukan "nilai", tapi **titik koordinat di peta milik si model** — dan tiap
model punya peta dengan sistem koordinat berbeda:

```
nomic menggambar "anggaran" di petanya → titik [0.2, -0.7, ...]
bge   menggambar "anggaran" di petanya → titik [0.9,  0.1, ...]

Makna sama, angka beda total — karena PETA-nya beda, bukan maknanya.
```

Kalau dicampur:

```
chunk lama : koordinat versi peta NOMIC   (masih tersimpan di db)
query baru : koordinat versi peta BGE     (model sudah diganti)

db membandingkan titik query vs titik chunk secara buta
→ seperti membandingkan koordinat peta Jakarta dengan peta Bandung:
  angkanya bisa dihitung, hasilnya keluar... tapi TIDAK BERMAKNA.
```

Bahayanya pola yang sama dengan filter salah: **db tidak protes.** Ia tidak tahu
vector lahir dari model mana; hasil query tetap keluar, tampak normal, padahal
semua jaraknya ngaco — silent garbage. Tiap model "hidup di alamnya sendiri";
angka jarak hanya bermakna antar penghuni alam yang sama.

Karena itu ganti model = semua chunk digambar ulang di peta baru, supaya query
dan chunk bicara dalam sistem koordinat yang sama.

## Riwayat

- 2026-07-20 — dibuat: 3 poin (bahasa ibu model → tes A/B variabel terkontrol → obat & re-index total) + analogi juru arsip/timbangan/masa jenis/peta + bukti lapangan MEMORY.md (diskusi R&D)
