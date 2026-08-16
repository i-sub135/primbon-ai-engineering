# RAG — Key Takeaways

Rangkuman poin penting dari sesi belajar RAG + fine-tuning.

## Fine-tune vs RAG
- Fine-tune = hafalan, bake ke weights. Update KB → harus retrain
- RAG = open-book, baca file saat query. Update KB → langsung efek
- Untuk "asisten KB" → RAG lebih tepat. Fine-tune untuk style/tone

## Chunk Strategy
- **By-size = salah** — potong tanpa peduli konteks → embedding ambigu
- **By-heading (`##`) = benar** untuk markdown KB — boundary semantik sudah lo define saat nulis
- PDF beda: format visual, parser reconstruct dari koordinat → unpredictable

## chunk_markdown() — mekanismenya
- `split('\n')` → array of lines
- Loop tiap baris, ketemu `##` → tutup chunk lama, mulai chunk baru
- Output: array of strings, tiap string = 1 section. Bukan per karakter

## embed() — mekanismenya
- Dipanggil **per chunk**, bukan per file
- 1 chunk → 1 vector (768 angka) = "posisi makna" di ruang vektor
- `nomic-embed-text` = embedding model (bukan LLM) — convert teks ke vector, bukan jawab pertanyaan

## Pipeline Dev Mode

    KB .md → chunk.json → chunk-embed.json → chromadb

- Tiap step bisa di-inspect sebelum lanjut
- Error gampang dilokasi → debug-ability
- Jangan build apa yang lo ga bisa inspect

## col.add() — mekanismenya

Dipanggil 1x per chunk. Simpan 4 hal sekaligus:
- `documents` — teks chunk asli (string)
- `embeddings` — vector hasil embed() (768 angka)
- `ids` — unique key: `source::hash(chunk)`
- `metadatas` — source file path

**Kenapa teks disimpan bareng vector:**
- Vector = untuk *nemuin* chunk yang relevan (similarity search)
- Teks = untuk *dibaca* model setelah ketemu
- Tanpa teks: chromadb ketemu tapi harus buka file lagi. Ga efisien.

**Vector search vs SQL LIKE:**
- `WHERE name LIKE '%LoRA%'` = match literal teks
- Vector search = match *makna*. Query "adapter fine-tune kecil" bisa return chunk yang nulis "LoRA" — kata beda, makna mirip

## Query Script — mekanismenya

Flow: pertanyaan → embed → col.query() → context → LLM jawab

### Aturan emas
- Query di-embed pakai model yang **SAMA** dengan indexer (`nomic-embed-text`).
  Beda model = beda "bahasa" vector → search ngaco total. Kesalahan klasik #1 di RAG.

### col.query() — bukan filter, tapi ranking
- Filter (WHERE) = saring cocok/ga cocok
- col.query() = bandingin q_vec ke SEMUA vector, urutkan by jarak, ambil top-N
- `n_results=3` → return 3 chunk terdekat, urut dari paling mirip

### Bentuk res

    res = {
      'documents': [["## LoRA\n...", "## Fine-tune\n...", "## Chunking\n..."]],
      'ids':       [["rag-overview.md::4821", ...]],
      'distances': [[0.12, 0.31, 0.78]],
    }

- `res['documents'][0]` — index [0] karena col.query() bisa terima banyak
  pertanyaan sekaligus; kita kirim 1, jadi ambil hasil pertanyaan pertama

### context — dari list jadi 1 string

    context = '\n\n'.join(res['documents'][0])

- Input: list 3 string (3 chunk)
- Output: 1 teks utuh, chunk ditempel berurutan dipisah baris kosong
- Bentuknya persis potongan KB disusun jadi "buku terbuka"

### Prompt akhir yang dilihat LLM

    [system] Jawab pakai context ini: <context>
    [user]   apa itu LoRA?

- Context di system message, pertanyaan asli di user message
- LLM jawab dengan buku terbuka, bukan dari hafalan weights

## Test End-to-End — urutan verifikasi

RAG bisa gagal di 2 lapisan berbeda:

    Lapisan 1: RETRIEVAL  → chunk yang balik salah/ga relevan
    Lapisan 2: GENERATION → chunk bener, tapi LLM jawabnya ngaco

Test langsung tanya-jawab = ga tau salahnya di mana. Pisah per lapisan:

    1. Cek isi db      → col.count() + col.peek()
    2. Test retrieval  → tanya, lihat chunk yang balik (TANPA LLM)
    3. Test generation → full flow, nilai jawabannya

### #1 Cek isi db
- `col.count()` → cocokkan dengan ekspektasi (10 file × ~5 section ≈ 50)
  - Terlalu kecil = indexer cuma makan sebagian file
  - Terlalu besar = kemungkinan duplicate dari re-index
- `col.peek(2)` → cek documents ada teksnya, embeddings ada angkanya

### #2 Test retrieval (TANPA LLM)
- Query → print top-3: distance + id + potongan teks
- Distance: makin KECIL makin mirip. Chunk jawaban harusnya di #1
- **Gold questions** — 5-10 pertanyaan yang udah tau harus kena chunk mana:

    "apa itu LoRA?"               → ## LoRA harus muncul #1
    "kenapa chunk by-size jelek?" → ## Chunk Strategy harus #1

- Semua kena #1 → retrieval sehat
- Banyak meleset → masalah di chunking/embedding, BUKAN di LLM
- Ketahuan tanpa perlu nyalain LLM = hemat waktu debug

Mindset sama dengan chunk.json/chunk-embed.json: inspect lapisan ini
dulu sebelum naik ke lapisan berikutnya.

### #3 Test generation (full flow)
- Retrieval udah terbukti sehat dari #2 → kalau salah di sini,
  pelakunya LLM atau prompt, bukan chunking/embedding
- Nilai jawabannya: datang dari chunk KB, atau LLM ngarang?

**2 mode gagal:**
- HALUSINASI — nambahin "fakta" yang ga ada di context
  (context: "~50-200MB" → jawab: "~500MB, rilis 2019 by Google")
- NGABAIKAN CONTEXT — jawab pakai pengetahuan umum dia,
  beda dengan isi KB

**Pertanyaan jebakan (senjata utama):**
- Tanya topik yang GA ADA di KB (misal kubernetes)
- Jawaban benar: "tidak ada di KB"
- Jawaban salah: dia jelasin lancar → ga disiplin ke context
  → jawaban lain juga ga bisa dipercaya

**Fix = system prompt, bukan ganti model:**

    Lemah: "Jawab pakai context ini: {context}"
    Kuat:  "Jawab HANYA berdasarkan context di bawah.
            Kalau jawabannya tidak ada di context,
            bilang 'tidak ada di KB' — JANGAN mengarang."

**Checklist:**
1. 5-10 gold questions → jawaban sesuai isi chunk
2. 2-3 pertanyaan jebakan → harus ngaku "ga ada di KB"
3. Gagal → tuning system prompt dulu, bukan ganti model

## Iterative Training

Materi iterasi fine-tune pindah ke rumahnya: [ai-tuning/layer-3-fine-tune.md](../ai-tuning/layer-3-fine-tune.md) bagian "Iterative Training".
