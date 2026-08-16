> **Bonus karya — salinan asli dari lumbung.** File ini di-copy apa adanya (plek-ketiplek) dari basis pengetahuan privat pemiliknya, atas izin pemilik, sebagai contoh hidup penerapan bab ini. Tautan internal di dalamnya menunjuk file yang tetap tinggal di lumbung dan sengaja tidak disertakan. Nol sensor diperlukan — sudah dipindai, tidak ada kredensial atau entitas sensitif.

# MOL — AI Tuning

Status: 8/17 selesai

## Fondasi

- [x] 1. Decision flow layer 0→3: selalu mulai dari layer paling ringan → [00-index.md](ai-tuning/00-index.md)
- [x] 2. Setup tools: ollama, Python, CUDA/Colab, unsloth → [layer-0-install-tools.md](ai-tuning/layer-0-install-tools.md)
- [x] 3. Model selection: size vs hardware, perbandingan family, mulai dari 1.5B → [model-selection-guide.md](ai-tuning/model-selection-guide.md)

## Layer ringan (tanpa GPU)

- [x] 4. Layer 1 — Modelfile: FROM/PARAMETER/SYSTEM, temperature vs top_p, num_ctx → [layer-1-modelfile.md](ai-tuning/layer-1-modelfile.md)
- [x] 5. Layer 2 — prompt engineering: zero-shot vs few-shot, 3 cara inject → [layer-2-prompt-engineering.md](ai-tuning/layer-2-prompt-engineering.md)

## Layer 3 — fine-tune (teori)

- [x] 6. Konsep LoRA/QLoRA + flow end-to-end: train → adapter → merge → GGUF → ollama → [layer-3-fine-tune.md](ai-tuning/layer-3-fine-tune.md)
- [x] 7. Dataset JSONL: coverage matrix, generate sintetis, split train/eval, red flags → [dataset-creation-guide.md](ai-tuning/dataset-creation-guide.md)
- [x] 8. Training di Apple Silicon: mlx-lm, alternatif tanpa CUDA → [training-mac-silicon.md](ai-tuning/training-mac-silicon.md)

## Belum dipelajari — praktik

- [ ] 9. Praktik pertama: 1 cycle training end-to-end di Colab T4 dengan dataset kecil
- [ ] 10. Membaca loss curve: train vs eval loss, deteksi overfit dari angka nyata
- [ ] 11. Eval before/after: bandingkan base model vs hasil tuning dengan test set yang sama
- [ ] 12. Iterasi training: dataset kumulatif antar round, versioning model (v1/v2, rollback) → [layer-3-fine-tune.md](ai-tuning/layer-3-fine-tune.md) bagian "Iterative Training"

## Belum dipelajari — lanjutan

- [ ] 13. Chat template: cara messages array dirender jadi token — sumber bug klasik antar family model
- [ ] 14. Catastrophic forgetting: model jago task baru tapi lupa kemampuan umum
- [ ] 15. Hyperparameter lanjutan: learning rate, batch size, scheduler
- [ ] 16. Preference tuning (DPO): mengajari model memilih jawaban lebih baik, bukan sekadar meniru
- [ ] 17. Kombinasi fine-tune + RAG: style/behavior dari tuning, knowledge dari retrieval — bersinggungan dengan [mol-rag.md](mol-rag.md)
