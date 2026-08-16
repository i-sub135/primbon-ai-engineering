# Layer 0 — Dependensi & Tools

Fondasi sebelum nulis code. Semua di bawah ini harus ✅ dulu, baru masuk layer 1 (chunking).
Status dicek langsung dari mesin i5, 2026-07-17 pagi.

## 1. Tools sistem

| Tool | Butuh | Status di i5 |
|---|---|---|
| Python | 3.10+ | ✅ 3.12.3 |
| git | any | ✅ repo `knowledge-bank` udah init |
| ollama | any | ✅ jalan |

## 2. Model ollama

| Model | Buat apa | Status |
|---|---|---|
| `nomic-embed-text` | embedding (teks → vector) | ✅ udah ke-pull (274 MB) |
| `gemma4:12b` | LLM generation (jawab pertanyaan) | ✅ udah ada (7.6 GB) |

Ga perlu pull apa-apa lagi. Kalau gemma4 kerasa berat pas testing, alternatif ringan: `ollama pull qwen3:4b`.

## 3. Python packages (pip)

Venv udah ada di `knowledge-bank/.venv` tapi masih KOSONG — baru pip doang.

```bash
cd ~/Documents/AI-TUNING/RAG/knowledge-bank
source .venv/bin/activate
pip install chromadb ollama
```

| Package | Buat apa |
|---|---|
| `chromadb` | vector database — nyimpen & nyari chunk |
| `ollama` | python client — manggil embedding + LLM lokal |

Nanti kalau udah masuk layer API (src/api): tambah `fastapi uvicorn`. Belum perlu sekarang.

## 4. Kunci ke requirements.txt

Habis install, langsung bekukan biar reproducible:

```bash
pip freeze > requirements.txt
```

## 5. Smoke test — bukti layer 0 beres

Jalanin 3 baris ini. Semua lolos = layer 0 kelar.

```bash
# 1. chromadb ke-import
python -c "import chromadb; print('chromadb OK')"

# 2. embedding jalan (harus keluar 3 angka float)
python -c "import ollama; print(ollama.embed(model='nomic-embed-text', input='halo')['embeddings'][0][:3])"

# 3. LLM jalan (harus keluar teks jawaban)
python -c "import ollama; print(ollama.generate(model='gemma4:12b', prompt='bilang halo')['response'])"
```

Gagal di #2/#3 → cek `ollama serve` jalan apa engga, bukan salah code lo.

## Checklist

- [ ] `pip install chromadb ollama` di dalam venv
- [ ] `pip freeze > requirements.txt`
- [ ] Smoke test 3 baris lolos semua
- [ ] Lanjut layer 1: `chunk_markdown()`
