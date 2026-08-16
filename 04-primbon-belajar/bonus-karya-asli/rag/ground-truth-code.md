# Ground Truth — Contoh Code

> Pasangan code untuk [ground-truth.md](ground-truth.md). Dokumen itu untuk **mikir**,
> dokumen ini untuk **kebayang**.
>
> **Konsep, alasan, dan rumus tidak diulang di sini** — semuanya di file konsep.
> Kalau rumusnya berubah, yang disentuh cuma file konsep, lalu code di sini disesuaikan.
> Jangan menaruh penjelasan konsep baru di dokumen ini.

Bahasa: Python. Contoh sengaja pakai nama file generik (`gizi.md`, `glosarium.md`) —
bukan nama file dari project mana pun, supaya tidak bisa disalin buta.

## Langkah 1 — Baca File JSONL

```python
import json

def baca_ground_truth(path):
    baris = []
    with open(path, encoding="utf-8") as f:
        for teks in f:
            teks = teks.strip()
            if not teks:                 # lewati baris kosong
                continue
            baris.append(json.loads(teks))
    return baris
```

Isi variabelnya setelah dijalankan:

```
gt = baca_ground_truth("ground-truth.jsonl")

gt[0] = {
    "tanya":   "mesin kasir itu apa sih",
    "patokan": ["glosarium.md#POS"],
    "jenis":   "parafrase",
    "catatan": "nol kata sama dengan chunk, uji makna",
}
len(gt) = 4
```

Catatan: `json.loads` memproses **satu baris**, bukan seluruh file. Kalau memakai
`json.load(f)` (tanpa `s`) file ini akan error — JSONL bukan satu objek besar.

## Langkah 2 — Validasi Patokan Sebelum Ngukur

```python
def cek_patokan_hilang(gt, patokan_di_store):
    hilang = []
    for baris in gt:
        for p in baris["patokan"]:            # baris jebakan: [] → loop tidak jalan
            if p not in patokan_di_store:
                hilang.append((baris["tanya"], p))
    return hilang
```

Isi variabelnya:

```
patokan_di_store = {"glosarium.md#POS", "gizi.md#Pola-Makan"}   # diambil dari vector store

hilang = cek_patokan_hilang(gt, patokan_di_store)
hilang = [("kenapa harus makan teratur", "lambung.md#Telat-Makan")]
```

Pemakaiannya berhenti di sini kalau ada yang hilang:

```python
if hilang:
    for tanya, p in hilang:
        print(f"PATOKAN HILANG: {p}   (dari: {tanya})")
    raise SystemExit("benerin ground truth dulu, jangan lanjut ngitung")
```

`patokan_di_store` dibentuk sebagai `set` supaya `in` cepat. Isinya diambil dari
metadata chunk di vector store, digabung jadi `"namafile#judul-heading"` —
persis bentuk yang dipakai di file ground truth.

## Langkah 3 — Hitung Per Baris

```python
def hitung_baris(patokan, hasil_topk):
    if not hasil_topk:              # retrieval tidak mengangkut apa pun
        return 0.0, 0.0             # tidak ada yang kena, tidak ada yang guna
    kena = len(set(patokan) & set(hasil_topk))     # irisan dua himpunan
    recall    = kena / len(patokan)
    precision = kena / len(hasil_topk)
    return recall, precision
```

Baris `if` pertama itu bukan hiasan. Retrieval **bisa** mengembalikan daftar kosong —
misalnya karena filter metadata terlalu sempit. Tanpa penjaga itu,
`kena / len(hasil_topk)` melempar `ZeroDivisionError`, dan yang jatuh bukan satu baris
tapi seluruh proses pengukuran.

Nol di situ benar secara makna: tidak ada yang diangkut → tidak ada yang kena,
tidak ada yang guna. Beda dengan baris jebakan, yang penyebutnya nol karena
**patokannya** kosong — itu tidak boleh dinilai nol, harus dipisah (langkah 4).

Isi variabelnya:

```
patokan    = ["gizi.md#Pola-Makan", "lambung.md#Telat-Makan"]
hasil_topk = [
    "gizi.md#Pola-Makan",
    "sarapan.md#Jam-Sarapan",
    "kopi.md#Lambung-Perih",
    "lambung.md#Telat-Makan",
    "olahraga.md#Pagi",
]

set(patokan) & set(hasil_topk) = {"gizi.md#Pola-Makan", "lambung.md#Telat-Makan"}
kena = 2

hitung_baris(patokan, hasil_topk) = (1.0, 0.4)
```

Fungsi ini **tidak menangani baris jebakan** — untuk `patokan = []`, `len(patokan)`
nol dan Python melempar `ZeroDivisionError`. Itu disengaja: pemisahan baris jebakan
dilakukan di langkah 4, bukan disembunyikan dengan `if` di dalam sini. Kalau
dinolkan di sini, baris jebakan ikut menarik rata-rata recall ke bawah —
padahal dia tidak seharusnya diukur pakai recall sama sekali.

## Langkah 4 — Kumpulkan, Pisahkan Baris Jebakan

```python
def kumpulkan(gt, ambil_topk, k):
    recall_semua, precision_semua = [], []
    jebakan_total, jebakan_lulus = 0, 0

    for baris in gt:
        hasil = ambil_topk(baris["tanya"], k)

        if not baris["patokan"]:                  # baris jebakan
            jebakan_total += 1
            if not hasil:                         # sistem tidak mengangkut apa pun
                jebakan_lulus += 1
            continue

        r, p = hitung_baris(baris["patokan"], hasil)
        recall_semua.append(r)
        precision_semua.append(p)

    return recall_semua, precision_semua, jebakan_total, jebakan_lulus
```

`ambil_topk` sengaja dikirim sebagai argumen, bukan dipanggil langsung di dalam.
Efeknya: fungsi ini bisa diuji tanpa menyalakan vector store —

```python
def topk_palsu(tanya, k):
    tabel = {
        "mesin kasir itu apa sih":     ["glosarium.md#POS", "laporan-q3.md#POS"],
        "definisi POS":                ["laporan-q3.md#POS", "jasa-kirim.md#POS"],
        "kenapa harus makan teratur":  ["gizi.md#Pola-Makan"],
        "cara setting kubernetes":     [],
    }
    return tabel.get(tanya, [])[:k]
```

Tabel palsu ini **wajib mencakup semua baris** di file ground truth. Kalau ada satu
pertanyaan yang tidak terdaftar, `tabel.get(tanya, [])` memberi daftar kosong —
dan itu terbaca sebagai "retrieval gagal total", padahal yang salah tabel palsunya.

Isi variabelnya, memakai empat baris contoh dari
[ground-truth.md](ground-truth.md) bagian Format File:

```
recall_semua    = [1.0, 0.0, 0.5]
precision_semua = [0.5, 0.0, 1.0]
jebakan_total   = 1
jebakan_lulus   = 1
```

Baris kedua nol dua-duanya — itu pertanyaan `"definisi POS"`, kasus satu-kata-dua-makna.
Patokannya `glosarium.md#POS`, tapi yang terangkat justru dua chunk POS yang lain.
Persis kegagalan yang ground truth dibuat untuk menangkap.

Catatan soal baris jebakan: contoh di atas menilai lulus kalau `hasil` kosong.
Di sistem nyata, "harusnya diam" biasanya diuji di lapis jawaban — apakah model
mengaku tidak tahu — bukan di lapis retrieval. Ganti isi `if` itu sesuai lapis
yang sedang diuji.

## Langkah 5 — Cetak Laporan Tiga Angka

```python
def rata(angka):
    return sum(angka) / len(angka) if angka else 0.0

def cetak_laporan(recall_semua, precision_semua, jebakan_total, jebakan_lulus, k):
    print(f"recall@{k}    = {rata(recall_semua):.2f}   ({len(recall_semua)} pertanyaan)")
    print(f"precision@{k} = {rata(precision_semua):.2f}   ({len(precision_semua)} pertanyaan)")
    print(f"jebakan       = {jebakan_lulus}/{jebakan_total} lulus")
```

Keluarannya:

```
recall@5    = 0.50   (3 pertanyaan)
precision@5 = 0.50   (3 pertanyaan)
jebakan       = 1/1 lulus
```

Angka di sini beda dengan contoh di [ground-truth.md](ground-truth.md) —
datanya beda (empat baris di sini, dua baris di sana). Yang harus sama cuma rumusnya.

Cacah pertanyaan dicetak di sebelah angkanya. Bukan hiasan: `0.75` dari 2 pertanyaan
dan `0.75` dari 30 pertanyaan bukan bukti yang sekuat, dan tanpa cacahnya
perbedaan itu tidak terlihat.

`rata()` mengembalikan `0.0` untuk daftar kosong supaya tidak error saat semua baris
ternyata jebakan. Nol di situ artinya "tidak ada yang diukur", bukan "nilainya nol" —
karena itu cacah pertanyaan harus selalu ikut tercetak.

## Menjalankan Semuanya

```python
K = 5

gt = baca_ground_truth("ground-truth.jsonl")

hilang = cek_patokan_hilang(gt, patokan_di_store)
if hilang:
    for tanya, p in hilang:
        print(f"PATOKAN HILANG: {p}   (dari: {tanya})")
    raise SystemExit("benerin ground truth dulu, jangan lanjut ngitung")

hasil = kumpulkan(gt, ambil_topk, K)
cetak_laporan(*hasil, k=K)
```

Urutannya mengikat: validasi berada **sebelum** `kumpulkan`, dan gagal keras
(`SystemExit`) — bukan sekadar peringatan yang bisa terlewat. Alasannya di
[ground-truth.md](ground-truth.md), bagian validasi.

## Riwayat

- 2026-08-07 — dibuat: pasangan code untuk ground-truth.md, 5 langkah + isi variabel tiap langkah, Python atas keputusan Iyan (diskusi R&D)
- 2026-08-07 — perbaikan: `hitung_baris` diberi penjaga untuk `hasil_topk` kosong (sebelumnya ZeroDivisionError); `topk_palsu` dilengkapi supaya mencakup keempat baris contoh. Semua angka di dokumen ini sudah dijalankan dan cocok dengan keluaran nyata
