# MOL — RAG

Status: 13 selesai, 12 belum

Aturan file ini:

- **Nomor cuma dikasih ke yang udah dicentang.** Urutannya baru jadi fakta setelah dipelajari.
- **Yang belum dikerjain gak bernomor** — cuma urut prioritas dari atas ke bawah. Posisinya boleh geser kapan aja.
- **Label `[Keluarga n/m]`** = item ini satu keluarga sama yang lain, dipecah karena kegedean. Bukan topik terpisah.
- Referensi dari dokumen lain: sebut namanya, jangan nomornya.

## Selesai

1. Konsep dasar: RAG vs fine-tune, kapan pakai yang mana → [rag-overview.md](rag/rag-overview.md)
2. Setup tools: chromadb + ollama + nomic-embed-text, smoke test → [dependencies.md](rag/dependencies.md)
3. Chunking: by-heading vs by-size, batas ~500 token → [rag-indexer.md](rag/rag-indexer.md)
4. Embedding: teks → vector, dipanggil per chunk → [rag-indexer.md](rag/rag-indexer.md)
5. Vector store: upsert, ID stabil (hashlib), persistence, path konsisten → [rag-indexer.md](rag/rag-indexer.md)
6. Retrieval: col.query = ranking bukan filter, top-k, distance → [rag-key-takeaways.md](rag/rag-key-takeaways.md)
7. Prompt augmentation: context di system prompt, disiplin ke context → [rag-key-takeaways.md](rag/rag-key-takeaways.md)
8. Testing 2 lapisan: retrieval (gold questions) vs generation (pertanyaan jebakan) → [rag-key-takeaways.md](rag/rag-key-takeaways.md)
9. Chunking lanjutan: overlap + token-based split untuk section panjang → [chunking-overlap.md](rag/chunking-overlap.md)
10. Metadata filtering: batasi search per source/folder saat query → [metadata-filtering.md](rag/metadata-filtering.md)
11. Embedding multilingual: uji query ID vs chunk ID/EN, alternatif model (mis. bge-m3) → [embedding-multilingual.md](rag/embedding-multilingual.md)
12. **[Eval formal 1/4]** Konsep: recall@k + precision@k, kenapa dipakai berdua → [eval-retrieval-metrics.md](rag/eval-retrieval-metrics.md)
13. **[Eval formal 2/4]** Ground truth: apa itu, aturan tulis-sebelum-tes, dipatok ke file+heading → [ground-truth.md](rag/ground-truth.md) (konsep) + [ground-truth-code.md](rag/ground-truth-code.md) (contoh code)

## Belum dikerjain — urut prioritas

- [ ] **[Eval formal 3/4]** Implementasi: hitung recall@k + precision@k beneran pakai gold questions, plus cara milih k — _prasyarat: tanpa angka, semua perbaikan di bawah gak bisa dinilai naik apa turun_
- [ ] **[Eval formal 4/4]** Metrik sensitif urutan (MRR / NDCG): recall & precision buta urutan, ini yang ngukur posisi jawaban di ranking — _dibutuhin buat menilai rerank dan context ordering_
- [ ] Retrieval lanjutan: hybrid search (keyword + vector), reranking — _perbaikan retrieval paling besar dampaknya, tapi butuh alat ukur dulu_
- [ ] Context ordering & lost-in-the-middle: urutan chunk di prompt itu keputusan
- [ ] Source citation: jawaban menyebut file sumber (pakai metadata `source`) — _murah, dampak praktis langsung kerasa_
- [ ] Re-index lifecycle: handling chunk terhapus — delete-then-reindex vs stale detection — _wajib begitu KB dipakai harian dan isinya berubah_
- [ ] Multi-turn RAG: rewrite pertanyaan lanjutan jadi pertanyaan mandiri sebelum retrieval
- [ ] Prompt injection via dokumen — _GATE: wajib ditutup SEBELUM RAG nyerap konten eksternal, bukan sesudah_
- [ ] Serving: expose RAG jadi API (FastAPI) + integrasi ke chat
- [ ] Semantic caching: cache jawaban untuk pertanyaan yang mirip
- [ ] GraphRAG: retrieval berbasis relasi antar-entitas, bukan cuma similarity
- [ ] Agentic RAG: LLM memutuskan sendiri kapan & apa yang di-retrieve

## Riwayat

- 2026-08-07 — eval formal dipecah jadi 4 (konsep / ground truth / implementasi / metrik urutan), ditandai sebagai satu keluarga
- 2026-08-07 — konvensi diubah: nomor cuma buat yang udah dicentang, yang belum cuma urut prioritas tanpa nomor
