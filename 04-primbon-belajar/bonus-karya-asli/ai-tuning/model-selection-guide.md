# Model Selection Guide

> Cara milih base model yang tepat untuk fine-tune lokal.

## Quick Decision

| Kebutuhan | Rekomendasi | Alasan |
|-----------|-------------|--------|
| Mulai, resource terbatas | `Qwen2.5-1.5B-Instruct` | Ringan, cukup cerdas, ~1GB GGUF |
| Bahasa Indonesia lebih kuat | `Qwen2.5-3B` / `Gemma-2-2B` | Lebih banyak data multilingual |
| Quality > size | `Qwen2.5-7B` / `Llama-3.1-8B` | Butuh >8GB VRAM |
| CPU / edge device | `Qwen2.5-0.5B` | Jalan tanpa GPU |

**Default starting point:** `unsloth/Qwen2.5-1.5B-Instruct` — sudah pre-optimized oleh unsloth, paling mulus di Colab T4.

## Size vs Hardware

| Model size | VRAM (QLoRA training) | VRAM (inference) | GGUF size (q4_k_m) |
|------------|----------------------|-------------------|---------------------|
| 0.5B | ~2GB | ~1GB | ~400MB |
| 1.5B | ~3GB | ~2GB | ~1GB |
| 3B | ~5GB | ~3GB | ~2GB |
| 7B | ~8GB | ~5GB | ~4GB |

Colab T4 = 15GB VRAM → aman untuk model sampai 7B dengan QLoRA.

## Family Comparison

| Family | Kelebihan | Catatan |
|--------|-----------|---------|
| **Qwen2.5** (Alibaba) | Multilingual kuat, instruction-tuned bagus | Kadang verbose |
| **Llama-3.x** (Meta) | Ecosystem luas, community besar | Bahasa non-English lebih lemah di size kecil |
| **Gemma-2** (Google) | Compact, efisien per parameter | Pilihan size lebih sedikit |
| **Mistral** | Fast inference | Dataset training tidak transparan |

## Kapan Naik ke Model yang Lebih Besar

Mulai dari 1.5B. Naik ke 3B kalau:
- Sudah fine-tuned tapi masih sering salah di edge case
- Training loss tidak turun setelah epoch 3 (model terlalu kecil untuk task)
- Output terlalu "polos" — kurang nuance untuk use case yang butuh pemahaman konteks

**Jangan langsung 7B untuk eksperimen pertama** — iterasi lebih lambat, Colab session lebih sering habis sebelum training selesai.

## HuggingFace vs Unsloth Hub

Unsloth punya mirror di HuggingFace dengan format `unsloth/[model-name]` yang sudah dioptimasi:
- Load lebih cepat
- Kompatibel langsung dengan `FastLanguageModel.from_pretrained()`
- Pilihan ada di: https://huggingface.co/unsloth

Pakai prefix `unsloth/` untuk training. Setelah merge → bebas pakai model base manapun.
