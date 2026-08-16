# Layer 3: Fine-tune (LoRA/QLoRA)

> Hasil eksplorasi 2026-07-15. Konteks: optimasi model lokal (ollama) untuk use case kasir POS (baby-kasir).
> Last resort. Coba layer 1 + 2 dulu. Butuh GPU + dataset.

## Kapan Fine-tune Worth It

- Layer 1 + 2 sudah dicoba dan masih ada gap
- Volume query tinggi → few-shot per-request mahal di latency/token
- Behavior terlalu spesifik untuk dijelaskan lewat contoh runtime doang

## Konsep LoRA

**Low-Rank Adaptation** — tidak mengubah weights asli. Nempel adapter kecil di atas weights yang dibekukan.

> Analogi: lo ga ngecat ulang tembok, lo nempel stiker di atas temboknya.

**QLoRA** = LoRA di atas base model yang dikuantisasi 4-bit → hemat VRAM, bisa jalan di Colab T4 gratis.

## Flow End-to-End

```
[1] kasir_dataset.jsonl
    (tiap baris = 1 contoh percakapan)
         ↓
[2] Upload ke Colab / taruh di folder lokal
         ↓
[3] Jalanin script Python (unsloth)
    ├── load base model dari HuggingFace (sekali download ~1–3GB)
    ├── inject LoRA adapter ke model
    ├── load dataset dari file .jsonl
    └── SFTTrainer.train() ← ini yang lama (30min–beberapa jam)
         ↓
[4] Output: adapter weights folder (~50–200MB)
    ├── adapter_model.safetensors  ← isi "patch"-nya
    ├── adapter_config.json        ← config LoRA (r, alpha, dll)
    └── tokenizer_config.json
         ↓
[5] Merge adapter + base model → satu file GGUF
    (format yang ollama mengerti)
         ↓
[6] ollama create kasir-v1 -f Modelfile
    (Modelfile cukup: FROM ./kasir-v1.gguf)
         ↓
[7] ollama run kasir-v1 "tadi laku aqua 2 dus"
    → eval before/after, compare hasilnya
```

**Yang paling penting:** adapter ≠ model baru. Base model tetap utuh. Adapter = delta perubahannya saja.

## Dataset JSONL

Format 1 baris = 1 contoh percakapan:

```jsonl
{"messages":[{"role":"system","content":"Kamu Kasir, asisten kasir."},{"role":"user","content":"tadi laku aqua 2 dus"},{"role":"assistant","content":"Kasir catat! Aqua 2 dus ya~"}]}
{"messages":[{"role":"system","content":"Kamu Kasir, asisten kasir."},{"role":"user","content":"rokok 3 bungkus sama korek 1"},{"role":"assistant","content":"Oke oke! Rokok 3 bungkus sama korek 1 ya, Kasir tulis!"}]}
{"messages":[{"role":"system","content":"Kamu Kasir, asisten kasir."},{"role":"user","content":"berapa harga aqua?"},{"role":"assistant","content":"Kasir belum tau itu, nanti Kasir tanya dulu ya!"}]}
```

**Yang wajib ada di dataset Kasir:**
- Transaksi normal (berbagai produk, satuan, cara nulis)
- Transaksi typo/singkatan (`"aq 2 ds"`, `"rkok 3"`)
- Pertanyaan yang Kasir tahu jawabnya
- Pertanyaan yang Kasir tidak tahu → harus jawab "belum tau"
- Input random/chitchat → Kasir tetap stay in character

**Jumlah minimal:** 100–150 baris, variasi per kelas merata.

## Script Unsloth

Unsloth = wrapper HuggingFace Transformers: 2x lebih cepat, 60% hemat VRAM, mulus di Colab T4.

```python
from unsloth import FastLanguageModel
from trl import SFTTrainer
from datasets import load_dataset

# Load base model
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/Qwen2.5-1.5B-Instruct",
    max_seq_length=2048,
    load_in_4bit=True,   # QLoRA — wajib di GPU <16GB
)

# Inject LoRA adapter
model = FastLanguageModel.get_peft_model(
    model,
    r=16,
    lora_alpha=16,
    target_modules=["q_proj", "v_proj"],
)

# Load dataset & train
dataset = load_dataset("json", data_files="kasir_dataset.jsonl")["train"]
trainer = SFTTrainer(
    model=model,
    train_dataset=dataset,
    dataset_text_field="messages",
    max_seq_length=2048,
    num_train_epochs=3,
)
trainer.train()
```

## Key Parameters

| Param | Fungsi | Nilai awal |
|-------|--------|-----------|
| `r` (rank) | Ukuran adapter. Makin besar = makin ekspresif, makin berat | 8–16 |
| `num_train_epochs` | Berapa kali baca ulang seluruh dataset. Makin banyak → risiko overfit | 2–3 |
| `load_in_4bit` | Quantize base model saat loading. Wajib di GPU <16GB | `True` |
| `lora_alpha` | Scaling factor adapter. Set = `r`, jarang diubah | = r |
| `target_modules` | Layer mana yang dipasangi adapter. Standar: q_proj + v_proj | default |

**Yang paling sering diutak-atik:** `r` dan `num_train_epochs`. Sisanya bisa default dulu.

## Adapter Weights Folder

Hasil output training — bukan model baru, cuma "delta" perubahannya:

```
output/
├── adapter_model.safetensors  ← isi patch-nya (yang penting)
├── adapter_config.json        ← config LoRA
└── tokenizer_config.json
```

**Delta** = selisih perubahan saja. Analoginya: lo punya foto asli (base model), lo edit dikit (training), delta = cuma bagian yang berubah — bukan foto baru dari nol. Makanya ukurannya kecil (~50–200MB) vs base model (~1–3GB).

## Risk: Overfit

Model hafal kalimat training, gagal generalize ke variasi input baru.

```
Dataset punya: "tadi laku aqua 2 dus"   → jawab benar ✓
Input baru:    "barusan jual aqua dua dus" → jawab aneh ✗
```

Model "hafal kalimat" bukan "ngerti maksud".

**Tanda overfit:** training loss turun terus, tapi eval loss malah naik → gap melebar.

**Pencegahan:** split train/eval set wajib sebelum training — biar bisa deteksi ini sebelum keburu di-deploy.

## Tools

- `unsloth` + `trl` (SFTTrainer) — training
- `llama.cpp` — convert adapter + model → GGUF
- Colab T4 (gratis, 15GB VRAM) — cukup untuk model 1.5B
- HuggingFace Hub — source base model (download sekali)

## GGUF

Format file khusus untuk model inference lokal. Analoginya: HuggingFace format = source code, GGUF = compiled binary. llama.cpp dan ollama hanya bisa baca GGUF.

## Merge: Simpel vs Manual

**Via unsloth (rekomendasi):**
```python
model.save_pretrained_merged(
    "kasir-merged",
    tokenizer,
    save_method="merged_16bit",
)
```
1 baris, merge + save selesai. Output di `./kasir-merged/` (atau path yang lo set di argumen pertama). Di Colab = `/content/kasir-merged/`.

**Cara manual via PEFT (lebih ribet, hasil sama):**
```
1. trainer.save_model()              → save adapter terpisah
2. Load ulang base model + adapter
3. model.merge_and_unload()          → merge via PEFT library
4. Save merged model
5. Convert ke GGUF via llama.cpp script terpisah
```

5 step vs 1 baris. Untuk Kasir, unsloth cukup.

**Convert ke GGUF:**
```bash
python convert_hf_to_gguf.py kasir-merged/ --outfile kasir-v1.gguf --outtype q4_k_m
```

## Iterative Training

- Train dari merged folder (HF format), bukan GGUF
- Dataset selalu cumulative: round baru = data lama + data baru. Jangan replace

## Quantization Type (`--outtype`)

Kode format: `q{bits}_{method}_{size}`

| Kode | Bits | Method | Size |
|------|------|--------|------|
| `q4_k_m` | 4-bit | k-quant | medium ← sweet spot |
| `q4_k_s` | 4-bit | k-quant | small |
| `q4_k_l` | 4-bit | k-quant | large |
| `q8_0` | 8-bit | standard | — (lebih besar, kualitas tinggi) |

- Makin kecil angka bit = makin kecil file, makin turun kualitas
- `k` = k-quant method: lebih cerdas milih layer mana yang dikuantisasi → kualitas lebih terjaga vs metode lama
- `q4_k_m` = paling umum dipakai, ~800MB–1.5GB untuk model 1.5B

**Pola ini standar resmi llama.cpp** — bukan kode random. Ada daftar lengkapnya di llama.cpp docs. Kalau `--outtype` tidak diset, default = `f16` (GGUF full precision, ukuran besar).

**Quantization terjadi di step CONVERT, bukan merge:**
```
save_pretrained_merged()              → merge adapter + base → folder HF format (~3GB, f16)
                                               ↓
convert_hf_to_gguf.py --outtype q4_k_m → convert + quantize → .gguf (~1GB)
```
`q4_k_m` cuma ngefek di step convert. Merge = gabungin weights doang, belum ada compress.
