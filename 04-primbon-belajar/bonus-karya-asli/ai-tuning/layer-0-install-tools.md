# Layer 0: Install Tools

> Pre-requisite sebelum mulai layer 1–3. Target: Linux & macOS.

## Overview

| Tool | Fungsi | Layer |
|------|--------|-------|
| `ollama` | Run model lokal + Modelfile | 1, 2 |
| `python` + `pip` | Runtime script training | 3 |
| `unsloth` | Fine-tune wrapper | 3 |
| `trl` | Training engine (SFTTrainer) | 3 |
| `datasets` | Load JSONL dataset | 3 |
| `llama.cpp` | Convert GGUF manual | 3 |

⚠️ **Fine-tune (Layer 3) = Linux + CUDA atau Colab.**
macOS: layer 1–2 jalan mulus. Training via unsloth tidak support macOS — gunakan Colab T4 (gratis).

---

## Linux

### Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama --version
```

### Python (conda — rekomendasi)

```bash
conda create -n llm-dev python=3.11
conda activate llm-dev
```

### CUDA (NVIDIA GPU)

```bash
nvidia-smi   # cek GPU dulu
# Install CUDA toolkit: https://developer.nvidia.com/cuda-downloads
```

Colab T4: CUDA sudah pre-install, skip ini.

### Training libs

```bash
pip install "unsloth[colab-new] @ git+https://github.com/unslothai/unsloth.git"
pip install trl datasets transformers accelerate
```

### llama.cpp (manual GGUF convert — opsional)

```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp && pip install -r requirements.txt
```

---

## macOS

### Ollama

```bash
brew install ollama
# atau download dari https://ollama.com/download/mac
```

Apple Silicon (M1/M2/M3): Metal backend otomatis — performa bagus untuk inference.

### Python

```bash
brew install --cask miniconda
conda create -n llm-dev python=3.11
conda activate llm-dev
```

### Libs (inference only — training tidak support macOS)

```bash
pip install transformers
pip install llama-cpp-python   # GGUF convert via Python
```

---

## Quick Check

```bash
ollama --version
python --version
nvidia-smi                      # Linux GPU
python -c "import unsloth"      # Linux only
```

---

## Catatan

- **Training**: Linux + NVIDIA GPU, atau Colab T4 (gratis, 15GB VRAM)
- **Inference**: Linux & macOS keduanya oke
- **Apple Silicon**: Metal untuk ollama, bukan pengganti CUDA untuk training
- **Colab**: environment paling mudah untuk mulai — CUDA, storage, semua pre-configured
