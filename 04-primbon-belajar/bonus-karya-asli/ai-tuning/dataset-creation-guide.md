# Dataset Creation Guide

> Panduan bikin dataset JSONL untuk fine-tune LLM lokal.

## Format Dasar

1 baris = 1 contoh percakapan. Format messages array:

```jsonl
{"messages":[{"role":"system","content":"[SYSTEM_PROMPT]"},{"role":"user","content":"[INPUT]"},{"role":"assistant","content":"[EXPECTED_OUTPUT]"}]}
```

## Coverage Matrix — Apa yang Harus Ada

Dataset bagus = coverage merata, bukan sekedar banyak baris.

| Kelas | Deskripsi | Porsi |
|-------|-----------|-------|
| Happy path | Input normal, lengkap, straightforward | 30–40% |
| Typo / singkatan | Input dengan kesalahan ketik umum | 20% |
| Edge case | Input ambigu, format aneh, qty di tengah kalimat | 15% |
| Out of scope | Pertanyaan yang model harus tolak / redirect | 15% |
| Chitchat / random | Input acak, model harus stay in character | 10% |

**Total minimal:** 100–150 baris. Makin sedikit coverage kelas, makin mudah overfit ke kelas mayoritas.

## Cara Generate: Sintetis via LLM

Cara paling efisien = generate pakai LLM besar (GPT/Claude), review manual per baris.

### Prompt template untuk generator

```
Generate 20 contoh percakapan dalam format JSONL untuk fine-tune model asisten [USE_CASE].

Rules:
- system: "[SYSTEM_PROMPT_SINGKAT]"
- user: variasi input (termasuk typo, singkatan, cara nulis berbeda)
- assistant: output yang diinginkan (konsisten formatnya)

Kelas yang harus ada:
- 8x [HAPPY_PATH]
- 4x typo/singkatan
- 4x pertanyaan yang model tahu jawabnya
- 4x pertanyaan yang model tidak tahu → jawab dengan redirect/disclaimer

Format output: 1 JSON object per baris, tidak ada komentar atau penjelasan tambahan.
```

Ganti `[USE_CASE]`, `[SYSTEM_PROMPT_SINGKAT]`, dan `[HAPPY_PATH]` sesuai domain lo.

### Review checklist per baris sebelum dipakai training

- [ ] Format JSON valid (tidak ada syntax error)
- [ ] `assistant` output konsisten dengan baris lain (format, tone, panjang)
- [ ] `user` input realistic — kayak yang beneran diketik pengguna asli
- [ ] Edge case ter-cover (typo yang sering terjadi di use case lo)
- [ ] Tidak ada contoh misleading — model belajar pola yang salah

## Split Train / Eval

Wajib sebelum training. Minimal 80/20:

```
100 baris total
├── 80 baris → train
└── 20 baris → eval (jangan dipakai sampai setelah training selesai)
```

Eval set = "tes akhir". Kalau eval dipakai untuk debugging selama training → datanya bocor, hasilnya tidak valid.

## Red Flags Dataset

- Semua `assistant` response panjangnya sama → model overfit ke panjang, bukan konten
- Tidak ada variasi di `user` input → model hafal kalimat, bukan ngerti maksud
- Semua kelas jumlahnya sama persis → susah detect mana yang underfitted
- Ada duplikat baris → model double-belajar hal yang sama, overfit lebih cepat
- `assistant` output terlalu panjang/verbose → model jadi verbose saat inference
