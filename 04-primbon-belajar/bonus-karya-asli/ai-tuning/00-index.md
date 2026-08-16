# AI Tuning — Index

> Hasil eksplorasi 2026-07-15/16. Panduan optimasi model lokal (ollama) dari layer ringan ke berat.

## File di Folder Ini

| File | Isi |
|------|-----|
| `layer-0-install-tools.md` | Install ollama, Python, CUDA, unsloth |
| `layer-1-modelfile.md` | Config model via Modelfile (no GPU) |
| `layer-2-prompt-engineering.md` | Few-shot, structured output, cara inject |
| `layer-3-fine-tune.md` | LoRA/QLoRA, training, merge, GGUF |
| `dataset-creation-guide.md` | Cara bikin JSONL dataset yang bagus |
| `model-selection-guide.md` | Cara milih base model yang tepat |
| `training-mac-silicon.md` | Fine-tune di Apple Silicon via mlx-lm (tanpa CUDA) |

## Decision Flow

```
Mulai dari sini
     ↓
Behavior bisa dideskripsikan lewat teks + param?
     ├── Ya → Layer 1 (Modelfile) — coba dulu
     └── Tidak → lanjut
             ↓
     Butuh format output konsisten atau few-shot?
          ├── Ya → Layer 2 (Prompt Engineering)
          └── Tidak → lanjut
                  ↓
          Layer 1+2 udah dicoba dan masih ada gap?
               ├── Ya → Layer 3 (Fine-tune)
               └── Tidak → balik ke Layer 1+2, cek lagi
```

**Prinsip: selalu mulai dari layer paling ringan.**
Fine-tune = last resort — mahal (GPU/waktu), risky (overfit), susah di-maintain.

## Layer Overview

| Layer | Nama | GPU? | Waktu setup | Kapan worth it |
|-------|------|------|-------------|----------------|
| 0 | Install Tools | - | 1x setup | Pre-requisite |
| 1 | Modelfile | Tidak | Menit | Persona, constraint, params |
| 2 | Prompt Engineering | Tidak | Jam | Format output, few-shot |
| 3 | Fine-tune | Ya | Hari | Behavior spesifik, volume tinggi |
