# 12-LEARNING — Map of Learning (MOL)

Folder untuk melacak progres belajar per topik.
Satu topik = satu pasangan: file `mol-<topik>.md` + folder `<topik>/`.

> **Scope:** folder ini berisi pemahaman & contoh kode saja — catatan hasil diskusi R&D.
> Bukan kode produksi. Contoh di sini applicable ke kode nyata, tapi implementasi
> sesungguhnya hidup di repo project masing-masing.

> **Note:** MOL adalah dokumen hidup — berkembang seiring waktu. Item checklist boleh bertambah, dipecah, atau direvisi seiring pemahaman bertambah. Selesai dicentang bukan berarti selesai selamanya.

## Apa itu MOL

- `mol-<topik>.md` = peta belajar — checklist learning path + ringkasan hal yang perlu dipelajari
- `<topik>/` = materi detail — catatan lengkap per item checklist
- Aturan pasangan: setiap file mol wajib punya folder topik, dan sebaliknya
- Aturan konteks: satu konsep dicatat di satu topik saja. Topik lain yang bersinggungan cukup memberi link, tidak menyalin
- Aturan riwayat: setiap file materi punya section `## Riwayat` di bagian bawah. Setiap edit menambah satu baris: `tanggal — apa yang berubah`

## Struktur

```
12-LEARNING/
├── readme.md
├── kamus-istilah.md      ← lintas topik, bukan milik satu MOL
├── mol-rag.md
├── rag/
├── mol-ai-tuning.md
└── ai-tuning/
```

## Kamus istilah

[`kamus-istilah.md`](kamus-istilah.md) duduk sejajar dengan file MOL, bukan di dalam folder topik —
istilahnya dipakai lintas topik. Tabelnya tiga kolom:

- **Istilah**
- **Arti** — arti harfiah versi standar, untuk mencocokkan dengan tulisan orang lain
- **Analogi** — versi yang lebih mudah dipahami; bukan definisi

Aturan: istilah masuk kamus begitu dipakai di sesi belajar. Analogi yang sudah pernah
dipakai saat diskusi harus dipakai ulang, jangan bikin analogi baru untuk hal yang sama.

## Cara membuat MOL baru

1. Buat folder `<topik>/`
2. Buat file `mol-<topik>.md` di level yang sama
3. Isi checklist: daftar rata (flat), urut nomor, checkbox `- [ ]`
4. Setiap item diberi link ke file detail di dalam folder topik
5. Item yang sudah dipahami diubah menjadi `- [x]`

## Format mol-*.md

```
# MOL — <Topik>
Status: n/total selesai

- [ ] 1. <konsep singkat> → [detail](<topik>/<file>.md)
- [ ] 2. ...
```
