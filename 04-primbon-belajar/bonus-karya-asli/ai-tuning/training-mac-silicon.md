# Training on Mac Silicon (mlx-lm)

> Alternatif untuk fine-tune di Apple Silicon (M1/M2/M3/M4) tanpa CUDA.
> Toolchain berbeda dari unsloth — jangan campur.

## Kenapa Mac Silicon Bisa

Apple Silicon punya **Unified Memory** — GPU dan CPU share RAM yang sama. Tidak ada bottleneck transfer data antara CPU RAM dan VRAM seperti di PC biasa.

| Spec | Kapasitas efektif |
|------|------------------|
| M1/M2 16GB | ~1.5B model dengan LoRA |
| M2/M3 Pro 32GB | ~3B–7B model dengan LoRA |
| M3 Max 64GB+ | ~13B+ model |

**Batasan:** Kecepatan training lebih lambat dari NVIDIA T4 (Colab). Untuk dataset besar (>1000 baris) atau model >3B → Colab lebih worth.

## Tool: mlx-lm

Framework Apple MLX dioptimasi native untuk Apple Silicon. Mendukung LoRA fine-tune, sama konsepnya dengan unsloth tapi pakai MPS (Metal Performance Shaders) bukan CUDA.

## Install

```bash
pip install mlx-lm
```

Tidak perlu CUDA, tidak perlu conda khusus. Python standard + pip cukup.

## Dataset Format

**Sama dengan unsloth** — JSONL dengan messages array:

```jsonl
{"messages":[{"role":"system","content":"[SYSTEM]"},{"role":"user","content":"[INPUT]"},{"role":"assistant","content":"[OUTPUT]"}]}
```

Taruh di folder, misalnya `./data/train.jsonl` dan `./data/valid.jsonl`.

## Training

```bash
mlx_lm.lora \
  --model mlx-community/Qwen2.5-1.5B-Instruct \
  --train \
  --data ./data/ \
  --iters 1000 \
  --batch-size 4 \
  --lora-layers 8 \
  --val-batches 25
```

| Flag | Fungsi | Default |
|------|--------|---------|
| `--model` | Base model dari mlx-community (HuggingFace mirror) | - |
| `--iters` | Jumlah iterasi training | 1000 |
| `--batch-size` | Berapa contoh per batch | 4 |
| `--lora-layers` | Berapa layer yang dipasangi LoRA (≈ rank di unsloth) | 8 |
| `--val-batches` | Iterasi validasi per checkpoint | 25 |

Output: folder `adapters/` berisi adapter weights.

## Merge + Convert ke GGUF

**Step 1 — Fuse adapter ke base model:**

```bash
mlx_lm.fuse \
  --model mlx-community/Qwen2.5-1.5B-Instruct \
  --adapter-path ./adapters/ \
  --save-path ./model-fused/
```

**Step 2 — Convert ke GGUF via llama.cpp:**

```bash
python convert_hf_to_gguf.py ./model-fused/ \
  --outfile my-model-v1.gguf \
  --outtype q4_k_m
```

**Step 3 — Create di ollama:**

```
FROM ./my-model-v1.gguf
PARAMETER temperature 0.7
```

```bash
ollama create my-model-v1 -f Modelfile
ollama run my-model-v1 "test input"
```

## Perbandingan: mlx-lm vs unsloth

| | mlx-lm (Mac) | unsloth (Linux/Colab) |
|---|---|---|
| Hardware | Apple Silicon | NVIDIA GPU / Colab T4 |
| Speed | Lebih lambat | 2x lebih cepat |
| Setup | Lebih simpel (pip only) | Lebih berat (CUDA setup) |
| Model support | mlx-community mirror | unsloth HF mirror |
| Output format | MLX adapter → GGUF | HF merged → GGUF |
| Hasil akhir | GGUF sama | GGUF sama |

## mlx-community

Mirror model di HuggingFace yang sudah dikonversi ke format MLX. Pakai prefix `mlx-community/` untuk model yang sudah siap pakai dengan mlx-lm.

Contoh: `mlx-community/Qwen2.5-1.5B-Instruct`, `mlx-community/Llama-3.2-3B-Instruct`

## Kapan Pakai Mac vs Colab

- **Mac:** Dataset kecil (<500 baris), model 1.5B, iterasi cepat, tidak mau setup Colab
- **Colab T4:** Dataset lebih besar, model >3B, butuh training lebih cepat, atau Mac RAM terbatas
