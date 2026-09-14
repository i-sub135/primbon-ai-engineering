# Bonus Karya Asli — Laci Catatan Keputusan dari Lumbung

Folder ini salinan **satu laci catatan keputusan (ADR) utuh** — 17 ADR bernomor + 4 catatan keputusan bertanggal — dari basis pengetahuan privat pemiliknya, disalin atas izin pemilik sebagai contoh hidup penerapan bab 03. Ini keputusan arsitektur beneran dari sistem yang jalan di produksi, lengkap dengan konteks, tradeoff, alternatif yang ditolak, dan seksi Push-back-nya — bukan contoh karangan.

**Yang diubah dari aslinya, demi segel:** (1) nama proyek internal disamarkan jadi "Chatbot Kasir"; (2) nama pemiliknya diganti "pemilik"; (3) nama produk model internal disamarkan jadi "kasir". Selebihnya utuh — termasuk path kode, angka threshold, dan tanggal keputusan.

**Soal tautan "Related"** di kepala tiap ADR: sebagian menunjuk dokumen proyek yang tinggal di lumbung dan **sengaja tidak diikutkan** (isi proyek = dapur, tidak diumbar). Tautannya dibiarkan biar terlihat cara ADR saling merujuk.

Mulai dari mana: `adr-012` (fallback dilarang bohong — paling universal), `adr-002` (batas wilayah AI), lalu `adr-009` + `adr-010` (sepasang keputusan yang saling menambal).

## Riwayat
- 2026-08-12 — salinan pertama, 17 dokumen. — Kasarung 6, kursi jilid 5 (model dipilih: Fable 5), penyusun.
- 2026-09-10 — disinkronkan ulang dari lumbung (clone git langsung): +4 ADR (014–017: plugin intent dispatcher, model intent, kuota chat harian, skema log LLM), dokumen lama disalin ulang; total 21 dokumen; penyamaran sama; pindaian sensor nol. — Kasarung 6, kursi jilid 5 (model dipilih: Fable 5), penyusun.
