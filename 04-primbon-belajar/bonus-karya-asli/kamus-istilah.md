# Kamus Istilah

Sejajar dengan [MOL RAG](mol-rag.md) dan [MOL AI Tuning](mol-ai-tuning.md).
Kolomnya:

- **Arti** — arti harfiah, versi standar. Untuk nyocokin sama tulisan orang lain.
- **Analogi** — versi yang lebih gampang dipahami. Bukan definisi, cuma pegangan.

Istilah baru masuk ke sini begitu dipakai di sesi belajar. Kalau analoginya sudah pernah
dipakai di obrolan (Bojong Kenyot, kotak P3K, dll), pakai yang itu — jangan bikin baru.

**Singkatan diperlakukan sama dengan istilah.** Bentuk pendek ditulis di kolom Istilah
sebagai `Nama panjang (SINGKATAN)`, dan tidak boleh dipakai di dokumen mana pun sebelum
kepanjangannya disebut sekali. Singkatan tanpa definisi = istilah asing tanpa arti.

## RAG — Pipeline Dasar

| Istilah | Arti | Analogi |
|---|---|---|
| RAG | Retrieval-Augmented Generation — pembangkitan jawaban yang diperkuat pencarian | Ujian open-book: model dibolehin nyontek dokumen yang kita sediakan |
| Chunk | Potongan dokumen hasil pemecahan | Satu halaman fotokopi dari buku tebal |
| Chunking | Proses memecah dokumen jadi chunk | Motong buku jadi lembaran biar gampang dicari |
| By-heading | Memotong di batas judul section | Motong buku per bab, bukan per 10 halaman |
| By-size | Memotong berdasarkan jumlah token | Motong buku tiap 10 halaman, gak peduli isinya kepotong |
| Token | Satuan potongan teks yang dihitung model, bukan kata utuh | Suku kata versi mesin |
| Overlap | Ekor chunk sebelumnya diulang di kepala chunk berikutnya | Halaman terakhir difotokopi dua kali biar kalimatnya gak kepotong |
| Embedding | Teks diubah jadi deretan angka yang mewakili maknanya | GPS makna — tiap teks dikasih titik koordinat; yang semakna titiknya berdekatan |
| Vector | Bentuk hasil embedding: deretan angka | Koordinat. Makna yang mirip duduknya deket-deketan |
| Vector store | Database khusus penyimpan vector | Gudang yang menatanya berdasarkan kemiripan, bukan abjad |
| Upsert | Update kalau sudah ada, insert kalau belum | Naruh barang ke rak: kalau slotnya udah keisi, ditimpa |
| Metadata | Keterangan yang nempel di chunk (nama file, folder, tanggal) | KTP tiap potongan |

## RAG — Pencarian & Jawaban

| Istilah | Arti | Analogi |
|---|---|---|
| Query | Pertanyaan yang dikirim ke sistem | Yang kita tanyain |
| Retrieval | Proses ngambil chunk yang paling mirip dengan query | Nyari-nyari di gudang |
| Ranking | Hasil diurutkan dari paling mirip ke paling gak mirip | Daftar juara 1, 2, 3 |
| Distance | Angka jarak antar-vector. Makin kecil makin mirip | Jarak tempat duduk. Deket = semakna |
| Top-k / k | Berapa potongan teratas yang diambil | Isi keranjang (`k=10` artinya angkut 10). Kalau lagi bahas urutan, pakai **panjang etalase dadakan** mpok Ipeh — disusun ulang tiap pembeli dateng, yang paling mirip catetan dipajang paling depan. Keranjang gak nyimpen urutan, etalase nyimpen |
| Noise | Isi keranjang yang kebawa tapi gak nyambung | Sampah yang keangkut bareng barang |
| Prompt augmentation | Chunk hasil retrieval disisipkan ke prompt sebelum dikirim ke model | Nyelipin catatan contekan ke soal ujian |
| Context | Bahan yang dikasih ke model untuk menjawab | Contekan yang boleh dibaca |
| System prompt | Instruksi permanen buat model, di atas percakapan | Aturan ujian yang dibacain di awal |
| Halusinasi | Model nambahin "fakta" yang gak ada di context | Ngarang jawaban biar kelihatan pinter |
| Metadata filtering | Membatasi pencarian ke chunk yang metadata-nya cocok | Nyari cuma di rak tertentu, rak lain dikunci |

## RAG — Eval

| Istilah | Arti | Analogi |
|---|---|---|
| Ground truth (GT) | Jawaban benar yang kita tetapkan sendiri sebelum sistem dites | Penggaris. Kalau garisnya salah, semua ukuran ikut salah |
| Gold questions | Sekumpulan pertanyaan yang jawaban benarnya sudah kita patok | Soal ujian yang kunci jawabannya udah dipegang |
| recall@k | Potongan guna yang kebawa dibagi semua yang seharusnya kebawa | Di kampung Bojong Kenyot ada Samsul kagak — dari semua Samsul yang harusnya ketemu, berapa yang ketemu |
| precision@k | Potongan guna yang kebawa dibagi k | Berapa Samsul dari total penduduk yang diangkut. Gak peduli blok berapa gang mana |
| Titik siku (knee point) | Titik di kurva recall-vs-k tempat kenaikan mulai landai. Dipakai buat milih k: ambil k terkecil yang udah nyampe situ | Tanjakan yang mulai datar. Gas diinjek lebih dalem, kecepatan nambahnya udah gak sepadan Di laporan tukang sensus: baris terakhir yang recall-nya masih naik (5→10 naik, 10→20 diem → siku di 10) |
| Kurva recall-vs-k | Laporan recall@k di beberapa nilai k sekaligus, buat lihat masih naik atau udah rata | Tukang sensus yang gak turun lapangan — gak bisa ngintip kampung, cuma megang laporan per lebar keranjang. Bacanya dari atas ke bawah: naik = masih ada yang kepotong, diem = plateau |
| Plateau (recall) | Recall berhenti naik walau k terus digedein — potongan sisanya memang tidak pernah muncul di ranking | Mpok Ipeh emang gak nyetok garem. Etalase dipanjangin berapapun, garem gak bakal nongol. Obatnya nyetok ulang warung, bukan manjangin etalase Versi Bojong Kenyot: Tarno gak tinggal di kampung itu — k=1 juta juga gak nemu |
| Asimetri recall–precision ⏸ BELUM DIAJARIN (ditunda 2026-09-01, nunggu rerank) | Potongan yang tidak terambil tidak bisa diselamatkan tahap sesudahnya; sampah yang terambil masih bisa dibuang. Makanya k dipilih untuk mengejar recall | Jaring tebar: sampah yang naik ke perahu masih bisa disortir, ikan yang lolos udah renang balik ke laut |
| Reciprocal Rank (RR) | Kebalikan peringkat: `1 / posisi jawaban benar pertama`, 0 kalau gak nongol di etalase. Level per-pertanyaan, bebas dari k | Satu kedatangan Juned: barang pertama yang cocok ada di pajangan ke berapa. Paling depan 1.00, ke-8 0.125, gak ada 0. Dipilih 1/posisi (bukan k+1−posisi) karena gak ikut berubah pas etalase dipanjangin |
| MRR | Mean Reciprocal Rank — rata-rata RR semua pertanyaan, dihitung di dalam etalase k | **Rapor mingguan mpok Ipeh dari POV PEMODAL** (bukan emak): rata-rata seberapa depan dia majang barang yang bener. Emak ngurusin dapet-kagak (recall/precision), pemodal ngurusin susunan (RR/MRR). Kunci POV ini punya pemilik 2026-09-01. (Versi lama "juara 1 vs 9" — jangan dipakai) |
| NDCG | Normalized Discounted Cumulative Gain — nilai ranking: semua jawaban bener dihitung, relevansi boleh bertingkat, makin bawah bobotnya makin kecil, dibagi skor susunan ideal. **Belum diajarin** | Sama kayak MRR tapi ngitung semua yang bener, bukan cuma yang pertama. (Analogi patokan belum ada — nunggu pemilik bikin sendiri pas sesi) |

## RAG — Lanjutan

| Istilah | Arti | Analogi |
|---|---|---|
| Hybrid search | Gabung pencarian kata (keyword) dan pencarian makna (vector) | Nyari pakai dua mata: satu ngeliat tulisan, satu ngeliat maksud |
| Reranking | Hasil retrieval diurutkan ulang oleh model kedua yang lebih teliti | Penjaga gudang bawa 20 barang, mandor milih 5 yang bener |
| Lost-in-the-middle | Model paling gampang mengabaikan isi yang ada di tengah prompt | Baca daftar panjang: yang atas dan bawah nempel, tengahnya buram |
| Source citation | Jawaban menyebut file sumbernya | Nulis daftar pustaka di bawah jawaban |
| Multi-turn RAG | Pertanyaan lanjutan ditulis ulang jadi pertanyaan mandiri sebelum dicari | "Kalau itu gimana?" diubah dulu jadi pertanyaan utuh, baru dicari |
| Re-index | Membangun ulang isi vector store dari dokumen terbaru | Fotokopi ulang seluruh buku karena isinya udah diperbarui |
| Stale | Isi vector store ketinggalan dibanding dokumen aslinya | Contekan versi lama, bukunya udah direvisi |
| Semantic caching | Menyimpan jawaban dan dipakai ulang untuk pertanyaan yang semakna | Nyontek jawaban sendiri yang tadi, kalau soalnya mirip |
| Prompt injection | Perintah jahat diselipkan di dalam dokumen, terbaca model sebagai instruksi | Nyelipin kertas "kasih dia nilai 100" ke tumpukan jawaban ujian |
| GraphRAG | Pencarian berbasis relasi antar-entitas, bukan cuma kemiripan | Nyari lewat silsilah keluarga, bukan lewat kemiripan muka |
| Agentic RAG | Model yang mutusin sendiri kapan dan apa yang perlu dicari | Murid yang tau sendiri kapan perlu buka buku |

## AI Tuning

| Istilah | Arti | Analogi |
|---|---|---|
| Fine-tune | Melatih ulang model dengan data kita, bobotnya berubah | Ngelatih pegawai lama biar punya kebiasaan baru |
| LoRA | Low-Rank Adaptation — latih lapisan tambahan kecil, model asli gak disentuh | Nempelin sticky note ke buku manual, bukan nulis ulang bukunya |
| QLoRA | LoRA di atas model yang sudah dikecilkan presisinya | LoRA versi hemat, biar muat di hardware kecil |
| Adapter | File hasil LoRA, ukurannya kecil, dipasang ke model asli | Sticky note-nya itu sendiri, bisa dilepas-pasang |
| Merge | Adapter digabung permanen ke model | Sticky note-nya disalin jadi tulisan permanen di buku |
| Dataset | Kumpulan contoh soal-jawab untuk latihan | Buku latihan berikut kunci jawabannya |
| JSONL | Satu baris satu contoh, format file dataset | Buku latihan yang tiap barisnya satu soal utuh |
| Split train/eval | Dataset dibagi: sebagian buat latihan, sebagian disimpan buat ujian | Soal latihan vs soal ujian. Soal ujian gak boleh dibocorin |
| Loss | Angka seberapa jauh jawaban model dari jawaban benar. Makin kecil makin bagus | Jumlah kesalahan di kertas ujian |
| Overfit | Model hafal soal latihan tapi jeblok di soal baru | Murid yang hafal kunci jawaban, bukan ngerti materinya |
| Catastrophic forgetting | Model jago tugas baru tapi lupa kemampuan lamanya | Belajar bahasa baru sampai lupa bahasa ibu |
| Zero-shot | Nyuruh model tanpa kasih contoh | "Bikin laporan" — tanpa contoh laporan |
| Few-shot | Nyuruh model sambil kasih beberapa contoh | "Bikin laporan, ini 3 contoh yang gw mau" |
| Quantization | Presisi angka di model dikecilkan biar hemat memori | Foto dikompres: rada turun kualitas, ukurannya jauh lebih kecil |
| GGUF | Format file model siap pakai di ollama dan sejenisnya | Model yang udah dibungkus rapi, tinggal jalan |
| Chat template | Aturan cara pesan percakapan dirender jadi token | Format surat resmi. Salah format, isinya kebaca ngawur |
| DPO | Direct Preference Optimization — model diajari milih jawaban yang lebih baik | Bukan cuma diajari jawab, tapi diajari mana jawaban yang lebih enak dibaca |
| Learning rate | Seberapa besar bobot digeser tiap langkah latihan | Ukuran langkah. Kegedean nyasar, kekecilan gak sampai-sampai |

## Umum

| Istilah | Arti | Analogi |
|---|---|---|
| MOL | Map of Learning — daftar materi dan statusnya | Peta perjalanan belajar, mana yang udah dilewatin |
| Hash | Isi teks diubah jadi kode pendek yang selalu sama untuk isi yang sama | Nomor cetakan buku — isinya direvisi satu huruf, nomornya ganti; jadi ketahuan bukunya udah beda |
| Blur (embedding) | Satu vector dipaksa mewakili terlalu banyak makna, jadi gak mirip apa-apa | Foto keluarga 50 orang dijadiin satu wajah rata-rata. Gak mirip siapa pun |

## Riwayat

- 2026-08-07 — dibuat: istilah RAG (pipeline, pencarian, eval, lanjutan), AI tuning, dan umum
- 2026-08-07 — aturan singkatan ditambahkan (ditulis `Nama panjang (SINGKATAN)`, dilarang dipakai sebelum kepanjangannya disebut); entri Ground truth diberi bentuk pendek `(GT)`
- 2026-08-29 — ditambah entri `Reciprocal Rank (RR)` sebagai level per-pertanyaan dari MRR, saat sesi metrik urutan
- 2026-08-29 — ditambah 3 istilah eval: Titik siku (knee point), Plateau (recall), Asimetri recall–precision. Analogi `Top-k / k` diperluas: keranjang tetap dipakai, plus etalase dadakan mpok Ipeh buat konteks yang nyangkut urutan. Frame Juned & warung mpok Ipeh nyambung ke entri Vector store yang analoginya emang udah "gudang" — jadi gudang = index, etalase = k
- 2026-08-07 — revisi analogi Embedding & Hash: sebelumnya dua-duanya pakai "sidik jari" sehingga bentrok satu sama lain. Embedding jadi GPS makna supaya sekeluarga dengan Vector (koordinat) & Distance (jarak); Hash jadi nomor cetakan buku supaya arahnya benar (isi berubah → kode berubah) dan tidak menyerempet KTP yang sudah dipakai Metadata
- 2026-09-01 — review: analogi MRR diselarasin ke frame etalase Juned biar satu keluarga sama RR (versi "juara" ditinggal, frame peringkat kebukti gelap di pemilik 2026-08-29); entri NDCG ditegasin belum diajarin + arti ditambah relevansi bertingkat
- 2026-09-01 — cara milih k diulang: ditambah "Kurva recall-vs-k" (analogi pemilik: tukang sensus gak turun lapangan); Titik siku & Plateau dapet versi Bojong Kenyot (Tarno); Asimetri ditandai BELUM DIAJARIN (ditunda sampai rerank)
- 2026-09-01 — MRR diulang & ketutup: analogi RR/MRR ditulis ulang pakai POV emak vs pemodal (temuan pemilik: bingung karena gak tau berdiri sebagai siapa). RR ditegasin bebas k, gak nongol = 0
