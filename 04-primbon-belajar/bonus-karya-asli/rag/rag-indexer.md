# RAG Indexer — Membangun Vector Store dari KB

> Script 1x-run: baca .md → split chunks → embed → simpan ke chromadb.
> Jalankan ulang setiap KB diupdate.

## Konsep: Chunking

Split dokumen jadi potongan kecil supaya embedding lebih fokus.

Strategi untuk markdown:
- **By heading (`##`)** — paling cocok, tiap section sudah semantically grouped
- By token count — potong tiap N token, overlap M token
- Hybrid — by heading, kalau section panjang potong lagi

Rule: chunk <500 token. Lebih dari itu = terlalu banyak noise saat search.

## Konsep: Embedding

Teks → vector angka. Chunks yang mirip maknanya → vector yang dekat.

    "LoRA adalah adapter kecil..."  → [0.12, -0.34, 0.89, ...]
    "Low-Rank Adaptation teknik..." → [0.11, -0.31, 0.91, ...]
                                       ↑ dekat! (makna mirip)

## Script
```python
    import os, hashlib, chromadb, ollama

    def chunk_markdown(text, source):
        chunks, current = [], []
        for line in text.split('\n'):
            if line.startswith('## ') and current:
                chunks.append('\n'.join(current))
                current = []
            current.append(line)
        if current:
            chunks.append('\n'.join(current))
        return [(c, source) for c in chunks if c.strip()]

    def embed(text):
        return ollama.embeddings(
            model='nomic-embed-text', prompt=text
        )['embedding']

    def chunk_id(src, chunk):
        # hashlib = stabil antar run; hash() bawaan Python di-random
        # per proses -> ID berubah tiap run -> duplicate saat re-index
        h = hashlib.md5(chunk.encode()).hexdigest()[:12]
        return f"{src}::{h}"

    def index_kb(kb_path, collection='knowledge-os'):
        kb_path = os.path.expanduser(kb_path)  # '~' tidak dipahami os.walk
        client = chromadb.PersistentClient(path='./chroma-db')
        col = client.get_or_create_collection(collection)

        n = 0
        for root, _, files in os.walk(kb_path):
            for f in files:
                if not f.endswith('.md'): continue
                path = os.path.join(root, f)
                text = open(path).read()
                for chunk, src in chunk_markdown(text, path):
                    col.upsert(              # upsert: ID ada -> update, baru -> insert
                        documents=[chunk],
                        embeddings=[embed(chunk)],
                        ids=[chunk_id(src, chunk)],
                        metadatas=[{"source": src}]
                    )
                    n += 1
        print(f"Done. {n} chunks indexed.")  # 0 chunks = path salah / kosong

    index_kb('~/Documents/Knowledge-OS/')
```
## Catatan Belajar: Apa yang Dipahami

### Chunking — cara kerjanya
- `split('\n')` → array of lines
- Loop tiap baris, kalau ketemu `## ` → tutup chunk sebelumnya, mulai chunk baru
- Output: **array of strings**, tiap string = satu section markdown
- Bukan per karakter, bukan per kata — per section semantik

### Embedding — cara kerjanya
- Tiap chunk → 1x panggil `embed()` → 1 vector (768 angka)
- 5 section di 1 file = 5x panggil embed() = 5 vector berbeda
- Vector = "posisi makna" di ruang angka

### Pipeline dev mode (recommended saat belajar)

    KB .md
      ↓ chunk_markdown()
    chunk.json          ← inspect: boundary bener ga?
      ↓ embed() per item
    chunk-embed.json    ← inspect: vector ada isinya?
      ↓ col.add()
    chromadb            ← source of truth

Tiap step bisa diverifikasi sebelum lanjut.
Kalau inline langsung, error susah dilokasi.

## Hal Penting

- **ID = `source::md5(chunk)[:12]`** — pakai `hashlib`, BUKAN `hash()` bawaan.
  `hash()` di-random per proses → ID beda tiap run → re-index bikin duplicate
- **`upsert()`, bukan `add()`** — add dengan ID existing di-skip, bukan di-update.
  Upsert + hash stabil = re-index aman (jalankan ulang → update otomatis)
- **Catatan re-index:** chunk yang DIHAPUS dari KB tetap nyangkut di db
  (upsert ga ngehapus). Kalau banyak hapus section → delete collection, index ulang
- **`os.path.expanduser()`** — `~` tidak dipahami `os.walk`; tanpa ini dapet
  0 file dan script "Done." tanpa error (silent failure)
- **Path db HARUS konsisten** — indexer & query script sama-sama `./chroma-db`.
  Beda path (misal `chroma_db` vs `chroma-db`) = query bikin db kosong baru
- **PersistentClient** — data tersimpan di disk, tidak hilang saat restart
- **Chunk boundary di `##`** — kalau section panjang >500 token, perlu potong lagi manual
