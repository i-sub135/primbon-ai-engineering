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
  - [x] Cara milih k (diulang & ketutup 2026-09-01) — dua sebab nama gak nongol, laporan multi-k (tukang sensus), plateau, titik siku, bantalan & harganya, k = garis potong. Asimetri kerugian DITUNDA sampai rerank → [eval-retrieval-metrics.md](rag/eval-retrieval-metrics.md) bagian "Cara Milih k"
  - [ ] Nulis code penghitungnya (kerjaan eksekusi, bukan kelas)
- [ ] **[Eval formal 4/4]** Metrik sensitif urutan (MRR / NDCG): recall & precision buta urutan, ini yang ngukur posisi jawaban di ranking — _dibutuhin buat menilai rerank dan context ordering_
  - [x] MRR (diulang & ketutup 2026-09-01) — RR = 1/posisi (bebas k, gak nongol = 0), Mean = level laporan, POV pemodal vs emak, kurva curam-di-depan + alasan desainnya, Mean sebagai level laporan → [eval-ranking-metrics.md](rag/eval-ranking-metrics.md)
  - [ ] NDCG — ngitung semua jawaban bener (bukan cuma yang pertama), relevansi bertingkat (gain), bobot mengecil ke bawah (log2, lebih landai dari RR), normalisasi. Butuh contoh motivasi baru: pasangan yang RR-nya sama
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
- 2026-08-29 — item 3/4 dipecah jadi dua sub-centang; "cara milih k" ditutup (materinya nebeng di eval-retrieval-metrics.md), sisa nulis code. Item belum dapat nomor karena belum tutup penuh
- 2026-08-29 — item 4/4 dipecah jadi dua sub-centang; MRR ditutup, file baru `rag/eval-ranking-metrics.md` lahir. Sisa NDCG
- 2026-09-01 — review: sub-item NDCG ditambah relevansi bertingkat (gain) + catatan contoh motivasi harus beda dari keranjang A vs B (itu udah kebedain MRR)
- 2026-09-01 — pemilik nyatain materi 08-29 (cara milih k + MRR) "keracunan": penjelasannya lompat, ambigu, nyender ke rerank/filter yang belum diajarin. Dua centang dicabut, status balik ke sebelum 08-29. Materi di KB TETAP disimpen sebagai arsip, tapi bukan bahan ngajar sampai diulang dari nol. Titik mulai ulang: cue Bojong Kenyot (recall/precision, 08-07) → trade-off k (08-17) → cara milih k
- 2026-09-01 — cara milih k diulang dari nol dan ketutup 19:52 (lolos cek-by-analogi, contoh Samsul/Udin/Tarno buatan pemilik). Asimetri kerugian dikeluarin dari sub-item, nunggu rerank. MRR masih dicabut
- 2026-09-01 — MRR diulang dari nol dan ketutup 21:25 lewat jalur Senin–Sabtu + POV pemodal. Eval formal 4/4 sisa NDCG
