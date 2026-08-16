# RAG — Retrieval Augmented Generation

> Pendekatan knowledge retrieval: model "baca" dokumen saat query,
> bukan hafal saat training. Lebih akurat, lebih mudah di-update.

## Kapan Pakai RAG vs Fine-tune

| | Fine-tune | RAG |
|---|---|---|
| Knowledge disimpan di | Weights model | File eksternal |
| Update konten | Harus retrain | Edit file, langsung efek |
| Akurasi faktual | Bisa hallucinate | Lebih akurat (baca langsung) |
| Cocok untuk | Style, behavior, tone | Knowledge retrieval |

## Flow

### Indexing (1x setup)
```
KB .md files
    ↓ split jadi chunks (per section/paragraf)
    ↓ convert ke vectors via embedding model
    ↓ simpan di vector DB
```

### Query (tiap pertanyaan)
```
user tanya
    ↓ pertanyaan di-convert ke vector
    ↓ cari chunks paling mirip di vector DB
    ↓ inject chunks ke prompt sebagai context
    ↓ LLM jawab berdasarkan context itu
```

## Komponen

| Komponen | Fungsi | Tool lokal |
|---|---|---|
| Embedding model | Teks → vector angka | `nomic-embed-text` (ollama) |
| Vector DB | Simpan + cari by similarity | `chromadb` |
| LLM | Jawab dari context | ollama model apapun |
| Orchestrator | Glue semua komponen | LangChain / LlamaIndex |

## Analogi

- **Fine-tune** = ujian hafalan. Semua harus dihafal sebelum ujian.
- **RAG** = ujian open-book. Boleh buka catatan saat menjawab.

KB lo = "buku" yang dibuka saat ada pertanyaan.

## Use Case: KB sebagai Knowledge Source

Langkah setup:
1. Index semua `.md` di `~/Documents/Knowledge-OS/`
2. Simpan vectors di chromadb
3. Saat query → retrieve chunks relevan → inject ke prompt

Update KB → langsung efek. Tidak perlu retrain.
