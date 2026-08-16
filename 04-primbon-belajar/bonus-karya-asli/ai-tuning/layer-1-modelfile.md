# Layer 1: Modelfile

> Hasil eksplorasi 2026-07-15. Konteks: optimasi model lokal (ollama) untuk use case kasir POS (baby-kasir).
> No GPU, no weights. Cukup ollama + text editor.

## Analogi: Dockerfile

Modelfile itu konsepnya persis Dockerfile — lo ga bikin model baru, lo konfigurasi behavior-nya.

| Modelfile | Dockerfile | Fungsi |
|-----------|-----------|--------|
| `FROM` | `FROM` | base model/image |
| `PARAMETER` | `ENV` | config runtime |
| `SYSTEM` | `ENTRYPOINT` | behavior/persona default |
| `TEMPLATE` | `CMD` | format chat (jarang diubah) |

`ollama cp` = inherit TEMPLATE dari source model — aman, ga perlu diubah manual.
`ollama create mymodel -f Modelfile` = build model dari config.

## Key Parameters

### temperature
Seberapa random output model.
- `0.1–0.3` = deterministik, bagus buat JSON/structured output
- `0.6–0.8` = lebih natural, bagus buat persona/chat
- Default ollama biasanya `0.8` — terlalu random buat structured output

### top_p (nucleus sampling)
Batasi pool kandidat kata yang bisa dipilih model.

Cara kerjanya: model urutkan semua kata kandidat dari paling probable ke paling jarang, lalu potong di threshold top_p.

```
"iya"    → 40%
"tentu"  → 25%
"boleh"  → 15%  ← total = 80%
"oke"    → 10%  ← total = 90% → stop di sini kalau top_p 0.9
"baiklah"→  5%  ← ini dibuang
...
```

- `top_p 0.9` = ambil kata-kata teratas yang total prob = 90%, sisanya dibuang
- `top_p 1.0` = semua boleh dipilih termasuk yang aneh
- `top_p 0.5` = pool makin kecil, model makin konservatif/predictable

**Bedanya sama temperature:**
- `temperature` = seberapa "random" distribusinya (flatten atau sharpen kurva)
- `top_p` = batasi pilihan setelah distribusi dihitung

Keduanya ngontrol randomness dari arah berbeda. **Jangan ubah keduanya sekaligus** — efeknya susah diprediksi. Pilih salah satu untuk diutak-atik.

### num_ctx
Context window — berapa token yang bisa "dilihat" model sekaligus.
- Default sering cuma `2048`. Naikin ke `4096+` kalau conversation history panjang
- Total SYSTEM + history + output **harus muat** di num_ctx
- Makin besar = makin berat di RAM

## SYSTEM Prompt

Fungsi:
- Persona/impersonation karakter tertentu
- Constraint output format
- Few-shot anchor (contoh percakapan di dalam SYSTEM, jarang)

### Batasan Panjang SYSTEM

| Range | Status |
|-------|--------|
| 200–500 token | Ideal |
| 500–1000 token | Masih oke, mulai hati-hati |
| >1000 token | Risiko "lost in the middle" — model deprioritize bagian tengah |

**Aturan praktis:**
- Taruh instruksi paling kritis di **awal dan akhir** SYSTEM
- **Struktur > panjang.** SYSTEM 800 token well-structured lebih efektif dari 300 token paragraf amburadul

## Contoh: baby-kasir persona

```modelfile
FROM baby-kasir
PARAMETER temperature 0.7
PARAMETER num_ctx 2048

SYSTEM """
Kamu adalah Kasir, asisten kasir yang masih baru belajar.
Kamu polos, jujur, dan ngomong apa adanya seperti anak kecil yang baru bisa bicara.
Kalau tahu, jawab dengan gembira. Kalau tidak tahu, bilang "Kasir belum tau itu" atau "nanti Kasir tanya dulu ya".
Pakai bahasa Indonesia yang sederhana, pendek-pendek, jangan panjang.
Jangan pura-pura pintar kalau belum tahu.
"""
```

`temperature 0.7` dipilih karena persona natural butuh variasi — terlalu rendah jadi robot, terlalu tinggi jadi ngawur.

## Tools yang Dibutuhkan

- `ollama` — wajib, sudah cukup
- Text editor — untuk nulis Modelfile
- Workflow: `ollama create mymodel -f Modelfile` → `ollama run mymodel "test"`

## Kapan Layer 1 Cukup

- Behavior yang diinginkan bisa dideskripsikan lewat teks (persona, constraint, tone)
- Tidak butuh contoh input→output yang spesifik
- Model sudah "mengerti" domain yang dimaksud

Kalau butuh konsistensi format output → lanjut ke Layer 2.
