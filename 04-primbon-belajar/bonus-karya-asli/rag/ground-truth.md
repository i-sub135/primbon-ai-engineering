# Ground Truth — Kunci Jawaban untuk Eval Retrieval

> Keluarga **Eval formal 2/4 — ground truth** di [MOL RAG](../mol-rag.md).
> Dokumen ini untuk **mikir**; contoh code-nya dipisah ke [ground-truth-code.md](ground-truth-code.md)
> supaya bisa dibaca tanpa melewati blok code.
> Prasyarat untuk ngitung [recall@k & precision@k](eval-retrieval-metrics.md).
> Lanjutan dari gold questions di [rag-key-takeaways.md](rag-key-takeaways.md).

## Apa Itu

```
Ground truth = kunci jawaban yang KITA tulis sendiri, sebelum sistem dites.
               Bukan output sistem. Bukan hasil ngukur.
               Dia yang jadi pembanding — bukan yang dibandingin.
```

Istilah `ground truth` <u>[kebenaran dasar — jawaban yang ditetapkan sendiri sebagai patokan,
bukan hasil tebakan sistem]</u>. Disingkat `GT` <u>[ground truth — dipakai sepanjang dokumen ini;
bukan singkatan lain]</u>.

Yang sering ketuker: ground truth **bukan angkanya**. `recall@k` dan `precision@k` itu
hasil ukur. Ground truth itu **penggarisnya**. Kalau penggarisnya salah, angkanya ikut
salah dan tidak ada yang memberi tahu.

Bentuknya sudah pernah dipakai tanpa disebut namanya:

```
Pertanyaan     : "kenapa harus makan teratur?"
Jawaban bener  : "makan yang teratur membuat sehat"   ← ini ground truth-nya
```

## Kenapa Perlu: Kasus Satu Kata Dua Makna

```
query: "definisi POS"

Pos Indonesia (BUMN)   ← ada kata POS
aplikasi kasir         ← ada kata POS, dan INI yang dituju
jasa pengantaran       ← ada kata POS
```

Ketiganya "nyambung" secara kata. Cuma satu yang benar. Tanpa ground truth,
sistem kelihatan pintar — tiga-tiganya relevan kok. Ground truth yang bikin dia ketangkap salah.

## Aturan Nomor Satu: Tulis SEBELUM Tes

```
BENER  : tulis "aplikasi kasir" dulu  →  jalankan retrieval  →  cocokkan
SALAH  : jalankan retrieval  →  lihat hasilnya  →  "oh iya yang kasir benar"
```

Yang salah itu mencocokkan penggaris ke barang. Nilainya akan selalu bagus,
karena tidak mungkin salah — kunci jawaban ditulis setelah melihat jawabannya.

Godaan ini muncul nyata saat ngoprek chunking dan hasilnya berubah:
*"ah ini juga benar sih sebenarnya"*. Sekali dibolehkan, semua angka kehilangan arti.

## Ground Truth Dipatok ke Apa

Ground truth harus bisa dicocokkan mesin, bukan dibaca manusia. Kalau isinya kalimat,
setiap pengecekan harus dibaca sendiri — tidak bisa 20 pertanyaan sekali jalan.

Tiga kandidat patokan dan kelemahannya:

```
ID chunk (hash isi)   → aman kalau urutan berubah
                      → JEBOL tiap isi chunk diedit (hash ganti)
nama file doang       → JEBOL karena 1 file bisa banyak pembahasan
judul heading doang   → JEBOL karena judul sama bisa ada di beda file
```

Contoh tabrakan heading:

```
glosarium.md    ## POS   ← definisi POS
laporan-q3.md   ## POS   ← angka penjualan per POS
```

Yang dipakai: **gandengan nama file + judul heading.**

```
GT = "glosarium.md#POS"
```

Nyambung ke keputusan chunking by-heading: karena batas chunk = heading,
heading itu memang nama alami tiap chunk. Bukan patokan baru, cuma pakai yang sudah ada.

### Kenapa heading, bukan ID

```
frekuensi jebol:  ID       tiap edit teks     (sering, tidak kerasa)
                  heading  tiap rename judul  (jarang, kerasa)
```

Heading bukan anti-jebol. Dipilih karena jebolnya jarang **dan kelihatan** —
saat rename judul, kita sadar sedang rename.

## Pertanyaannya Diambil dari Mana

Dua sumber pertanyaan, kerjanya beda — dua-duanya perlu, bukan pilih satu:

```
A = tes ALAT       "pipeline gw jalan gak?"
    pertanyaan dikarang dari isi KB, jawabannya dijamin ada
    gagal → code cacat

B = tes KENYATAAN  "sistem gw guna gak buat orang?"
    pertanyaan asli dari pengguna, jawabannya tidak dijamin ada
    gagal → bisa jadi KB-nya yang kurang, bukan code
```

Urutannya A dulu. Kalau A gagal, ngukur B tidak ada gunanya — mengukur dengan alat rusak.

### Jebakan di A: nyontek kalimat chunk

```
isi chunk : "POS = point of sale, aplikasi kasir untuk transaksi ritel"
GT ditulis: "apa itu point of sale untuk transaksi ritel"   ← comot frasanya
recall    : 1.00   mulus, tapi tidak menguji apa-apa
```

Pertanyaan yang memakai kata-kata dari chunk itu sendiri sudah pasti dekat jaraknya.
Yang terbukti hanya: mesin bisa menemukan teks yang disalin dari dirinya sendiri.

Pertanyaan pengguna sesungguhnya tidak begitu:

```
"mesin kasir itu apa sih"
   → tidak ada kata "POS", "point of sale", maupun "ritel". Nol kata sama.
```

Di situ baru terlihat retrieval benar-benar mengerti makna atau cuma cocok-cocokan kata.

```
BENER : tulis pertanyaan pakai bahasa orang luar, jangan lihat kalimat chunk-nya
SALAH : buka chunk, comot frasanya, jadikan pertanyaan
```

Cara praktis: tutup dulu KB-nya, tulis pertanyaan dari kepala, baru buka KB untuk
menentukan patokannya menunjuk mana.

## Ground Truth Boleh Kosong

Baris tanpa patokan bukan baris rusak — itu kasus uji yang sah, yaitu pertanyaan jebakan:

```
pertanyaan : "definisi POS"             patokan : glosarium.md#POS
pertanyaan : "mesin kasir apa sih"      patokan : glosarium.md#POS
pertanyaan : "cara setting kubernetes"  patokan : (kosong)
```

Baris kosong menguji sistem saat dia **harusnya diam**. Tanpa itu, yang teruji cuma
keadaan saat sistem bisa menjawab.

Cara menilainya beda, dan ini penting:

```
baris berpatokan : dinilai pakai recall@k / precision@k
baris kosong     : TIDAK bisa pakai recall — penyebutnya nol, angkanya tidak berarti
                   lulus  = sistem bilang "tidak ada di KB"
                   gagal  = sistem menjelaskan dengan lancar
```

Karena satu file GT berisi dua jenis baris dengan cara nilai berbeda, formatnya nanti
perlu menandai jenis tiap baris.

## Berapa Banyak, dan Campurannya

Jumlah menentukan apakah angkanya bisa dipakai mengambil keputusan:

```
5 pertanyaan   → 1 gagal = recall turun 0.20   berisik
20 pertanyaan  → 1 gagal = recall turun 0.05   mulai stabil
```

Dengan 5 pertanyaan, satu perubahan nasib bikin angka melompat 20% — tidak bisa
dibedakan "perbaikan berhasil" dari "kebetulan".

```
5–10    smoke test    cuma untuk "pipeline jalan gak", bukan untuk memutuskan
20–30   bisa dipakai   naik-turunnya mulai berarti
50+     nyaman         tapi biaya menulisnya naik
```

Campurannya jangan semua gampang:

```
sebagian    parafrase    bahasa lain, jawabannya ada     ← porsi terbesar
sebagian    ambigu       satu kata dua makna (kasus POS)
beberapa    jebakan      patokan kosong
beberapa    lintas file  jawabannya butuh 2+ chunk
```

Dan sebarannya harus merata ke seluruh KB. Kalau 20 pertanyaan semuanya menyasar 3 file
yang sama, yang teruji hanya sudut kecil KB — sisanya tidak pernah tersentuh, dan
kerusakan di sana tidak akan pernah terlihat.

## Format File

Isinya tinggal merangkum keputusan di atas — empat field:

```
tanya      teks pertanyaan
patokan    daftar "namafile#judul-heading", boleh kosong
jenis      parafrase / ambigu / jebakan / lintas-file
catatan    kenapa baris ini dibuat — untuk manusia, bukan mesin
```

Field `jenis` wajib ada karena baris jebakan dinilai berbeda; tanpa penanda,
script tidak tahu baris mana yang tidak boleh dihitung pakai recall.

Field `catatan` bukan hiasan: tiga bulan kemudian, baris yang gagal tidak akan terbaca
lagi apakah memang sengaja dibuat susah atau salah tulis. Satu kalimat cukup.

### Wadahnya: JSONL

Tiga kandidat yang ditimbang:

```
CSV    [Comma-Separated Values — nilai dipisah koma, seperti Excel mentah]
       + paling ringkas
       - patokan bisa lebih dari satu → dijejalkan ke satu sel, jadi kotor
       - tidak bisa dikomentari

JSONL  [JSON Lines — satu baris satu objek utuh]
       [JSON = JavaScript Object Notation, format tukar data pakai {} dan []]
       + satu baris satu kasus, mudah dibaca mesin
       + daftar patokan natural: ["a.md#X", "b.md#Y"]
       - tidak bisa dikomentari, dan menulis tangan agak berisik tanda baca

YAML   [YAML Ain't Markup Language — format teks pakai indentasi, bukan tanda baca]
       + paling nyaman ditulis tangan
       + bisa dikomentari pakai #
       - salah spasi = rusak, dan tidak langsung terasa
```

**Dipilih JSONL** (keputusan Iyan): sudah familiar, mudah di-update, dan ramah script generator.

Dua keuntungan tambahan:

- **Ramah git.** Satu baris satu kasus → ganti satu pertanyaan = satu baris berubah di diff.
  Di YAML, satu edit bisa terlihat mengubah blok.
- **Kelemahan "tidak bisa dikomentari" gugur**, karena keterangan sudah jadi field `catatan` —
  ikut terbaca mesin, tidak hilang saat file diproses ulang. Lebih baik daripada komentar.

### Bentuk Konkret

```json
{"tanya":"mesin kasir itu apa sih","patokan":["glosarium.md#POS"],"jenis":"parafrase","catatan":"nol kata sama dengan chunk, uji makna"}
{"tanya":"definisi POS","patokan":["glosarium.md#POS"],"jenis":"ambigu","catatan":"POS juga muncul di laporan-q3 dan jasa kirim"}
{"tanya":"kenapa harus makan teratur","patokan":["gizi.md#Pola-Makan","lambung.md#Telat-Makan"],"jenis":"lintas-file","catatan":"jawaban benar butuh dua chunk"}
{"tanya":"cara setting kubernetes","patokan":[],"jenis":"jebakan","catatan":"tidak pernah ditulis di KB, harus mengaku tidak tahu"}
```

Isi tiap field pada baris pertama:

```
tanya    "mesin kasir itu apa sih"      teks, satu pertanyaan
patokan  ["glosarium.md#POS"]           daftar: satu isi, banyak isi, atau [] kosong
jenis    "parafrase"                    salah satu dari empat label
catatan  "nol kata sama dengan chunk"   untuk manusia
```

### Aturan: `patokan` Selalu Daftar

```
BENER : "patokan":["glosarium.md#POS"]      selalu daftar, walau isinya satu
SALAH : "patokan":"glosarium.md#POS"        kadang teks, kadang daftar
```

Bentuk yang berubah-ubah memaksa script memeriksa dua kemungkinan tiap baris.
Bentuk seragam = kode pembacanya satu jalur, tanpa percabangan.

Baris jebakan cukup `[]` — daftar kosong. Bukan `null`, bukan field yang dihilangkan.

## Langkah yang Gampang Kelewat: Validasi GT Dulu

Sebelum ngitung metrik, cek semua patokan di ground truth masih ada di database.

Kalau ada yang hilang, gejalanya jahat: **tidak ada error**. Retrieval jalan normal,
hasilnya wajar, hanya tidak ada yang cocok dengan patokan di kunci jawaban.
Recall jeblok, lalu chunking disangka rusak — padahal yang rusak penggarisnya.

```
1. baca semua patokan di ground truth
2. cek satu-satu masih ada di vector store
3. ada yang hilang → benerin ground truth-nya DULU
4. baru ngitung recall@k / precision@k
```

## Rumus

Tiga rumus, satu tempat. **Kalau ada revisi, revisi di sini** — dokumen lain
(termasuk contoh code) menyesuaikan ke blok ini, bukan menyimpan salinannya sendiri.

```
kena     = cacah patokan yang ketemu di hasil top-k
harusnya = cacah patokan di baris ground truth
diangkut = k

recall@k     = kena / harusnya          per baris
precision@k  = kena / diangkut          per baris

recall@k sistem     = rata-rata recall@k semua baris BERPATOKAN
precision@k sistem  = rata-rata precision@k semua baris BERPATOKAN
angka jebakan       = jebakan yang lulus / total baris jebakan
```

Baris jebakan (`patokan: []`) punya `harusnya = 0`, jadi tidak boleh masuk dua rata-rata
pertama — pembagian nol. Dia punya angkanya sendiri, yang ketiga.

## Cara Gunain: Dari File Sampai Keluar Angka

> Contoh code lengkap per langkah: [ground-truth-code.md](ground-truth-code.md).
> Di bawah ini alur dan angkanya saja.

Lima langkah, urut:

```
1. baca file ground truth
2. validasi semua patokan masih ada di vector store   ← lihat section di atas
3. per baris: jalankan retrieval, ambil top-k
4. cocokkan hasil vs patokan → angka PER BARIS
5. rata-ratakan lintas baris → angka SISTEM
```

Langkah 4 dan 5 sering tercampur. Langkah 4 memberi angka **per pertanyaan**;
langkah 5 yang memberi angka **sistem**.

### Satu Baris Dijalankan (k = 5)

```
baris ground truth:
  tanya   : "kenapa harus makan teratur"
  patokan : ["gizi.md#Pola-Makan", "lambung.md#Telat-Makan"]
  jenis   : "lintas-file"
```

Hasil retrieval top-5:

```
1. gizi.md#Pola-Makan          ← cocok patokan
2. sarapan.md#Jam-Sarapan
3. kopi.md#Lambung-Perih
4. lambung.md#Telat-Makan      ← cocok patokan
5. olahraga.md#Pagi
```

Hitungannya:

```
kena     = 2      patokan yang ketemu di top-k
harusnya = 2      panjang daftar patokan
diangkut = 5      k

recall@5    = 2/2 = 1.00
precision@5 = 2/5 = 0.40
```

Baris kedua yang setengah gagal — patokan sama, tapi hanya satu yang terangkat:

```
recall@5    = 1/2 = 0.50
precision@5 = 1/5 = 0.20
```

### Langkah 5: Rata-rata Lintas Baris

```
recall@5    = (1.00 + 0.50) / 2 = 0.75
precision@5 = (0.40 + 0.20) / 2 = 0.30
```

### Baris Jebakan Dipisah dari Rata-rata

Baris berpatokan `[]` punya "harusnya" = 0 → pembagian nol. Tidak bisa masuk rata-rata recall.
Baris itu dihitung dengan cara sendiri:

```
recall & precision   dari baris berpatokan saja
angka jebakan        berapa dari N jebakan yang benar-benar mengaku "tidak ada di KB"
```

Jadi laporan akhirnya **tiga angka**, bukan dua.

### Angka Ditulis Desimal

Bukan soal benar-salah, soal bisa dibandingkan: `2/3` vs `5/8` tidak langsung terbaca
mana yang lebih besar; `0.67` vs `0.63` langsung terbaca.

### Kenapa "Cara Milih k" Jadi Bahasan Sendiri

```
recall@k    → k digedein: NAIK atau tetap
precision@k → k digedein: TURUN
```

Dua angka jelek tidak bisa diperbaiki sekaligus hanya dengan menggeser `k` —
arah obatnya berlawanan. Itu keputusan tersendiri, masuk item implementasi di MOL.

## Key Point

Yang wajib kebawa dari dokumen ini:

1. Ground truth = **penggaris**, bukan hasil ukur. Penggaris salah → semua angka salah, tanpa peringatan.
2. **Tulis sebelum tes.** Menentukan jawaban benar setelah melihat hasil = mencocokkan penggaris ke barang; nilainya pasti bagus dan pasti tidak berarti.
3. Patokannya **nama file + judul heading**, bukan ID hash dan bukan kalimat. File saja tidak unik, heading saja bisa tabrakan antar file.
4. Heading dipilih bukan karena anti-jebol, tapi karena **jebolnya jarang dan kelihatan**. ID jebol tiap edit teks dan tidak terasa.
5. **Validasi GT sebelum ngukur.** Patokan yang menunjuk chunk hilang tidak memunculkan error — cuma bikin recall jeblok dan chunking disangka rusak.
6. Sumber pertanyaan ada dua: A (dikarang dari KB, tes alat) dan B (pertanyaan asli, tes kenyataan). **A dulu**, karena B tidak berguna kalau alatnya rusak.
7. Saat menulis A, **jangan comot frasa dari chunk**. Tutup KB, tulis dari kepala, baru buka KB untuk menentukan patokan.
8. **Patokan kosong itu sah** — itu pertanyaan jebakan. Dinilai beda: lulus kalau sistem mengaku tidak tahu. Tidak bisa pakai recall (penyebutnya nol).
9. **5–10 pertanyaan cuma smoke test.** Untuk mengambil keputusan butuh 20–30, supaya satu kegagalan tidak menggoyang angka 20%.
10. Campur jenisnya (parafrase, ambigu, jebakan, lintas file) dan **sebar merata ke seluruh KB** — bukan menumpuk di beberapa file yang sama.
11. Wadahnya **JSONL**, empat field: `tanya`, `patokan`, `jenis`, `catatan`. Satu baris satu kasus → ramah git.
12. **`patokan` selalu daftar**, walau isinya satu; baris jebakan pakai `[]`. Bentuk seragam bikin kode pembacanya satu jalur.
13. Alurnya **5 langkah**; yang gampang tercampur: langkah 4 = angka per pertanyaan, langkah 5 = angka sistem.
14. Laporan akhir **tiga angka**: recall, precision, dan angka jebakan — bukan dua.
15. Recall & precision **obatnya berlawanan arah** terhadap `k`. Itu sebabnya "cara milih k" jadi bahasan sendiri.

## Kenapa Konsep & Cara-Set Tidak Ada Contoh Code

Ini disengaja, bukan terlewat:

```
konsep     recall/precision itu ide, bukan mekanik. Code menutupinya — pembaca
           jadi membaca sintaks, bukan logika. Yang dibutuhkan: rumus + contoh angka.
cara set   "set" itu kerjaan tangan, bukan kerjaan mesin. Artefaknya file JSONL,
           dan contoh JSONL-nya sendiri sudah berbentuk code.
cara pakai mekanik murni dan langkahnya kaku → di sinilah code paling jelas.
           Ada, dipisah ke ground-truth-code.md
```

Jangan menambahkan contoh code ke dua bagian pertama tanpa alasan baru yang jelas.

## Status & Sisa Utang

- Konsep, aturan waktu, pilihan patokan: **ketutup**
- Sumber pertanyaan, patokan kosong, jumlah & komposisi: **ketutup**
- Format file (JSONL + empat field): **ketutup**
- Cara gunain (5 langkah + hitung per baris + rata-rata): **ketutup**
- Cara milih k: **belum** — item "Eval formal — implementasi" di MOL
- Metrik sensitif urutan (MRR / NDCG): **belum** — item terpisah di MOL

## Riwayat

- 2026-08-07 — dibuat: definisi, kasus satu-kata-dua-makna (contoh POS punya Iyan), aturan tulis-sebelum-tes, pilihan patokan file+heading, validasi GT sebelum ngukur (diskusi R&D)
- 2026-08-07 — ditambah: section Rumus (satu tempat, jadi sumber tunggal); contoh code dipisah ke [ground-truth-code.md](ground-truth-code.md) atas usul Iyan (konsep = mikir, code = kebayang); ditambah alasan kenapa konsep & cara-set sengaja tanpa code
- 2026-08-07 — ditambah: cara gunain — 5 langkah alur, contoh hitung per baris (k=5), rata-rata lintas baris, baris jebakan dipisah dari rata-rata, laporan tiga angka; key point jadi 15 poin. Dokumen ini menutup keempat pertanyaan awal Iyan (di mana / apa itu / cara set / cara gunain)
- 2026-08-07 — ditambah: format file — empat field (tanya/patokan/jenis/catatan), wadah JSONL atas keputusan Iyan (familiar + ramah script generator + ramah git), aturan `patokan` selalu daftar; key point jadi 12 poin
- 2026-08-07 — ditambah: sumber pertanyaan A vs B + jebakan nyontek frasa chunk, ground truth boleh kosong (pertanyaan jebakan) dan cara nilainya yang beda, jumlah & komposisi & sebaran; plus section Key Point (diskusi R&D)
