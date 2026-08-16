---
aliases:
  - "Chat Engine - Intent Classification Contract Fix"
tags:
  - chatbot-kasir
  - chat-engine
  - intent-routing
  - llm
  - architecture
  - decision
status: decided-ready-for-handoff
created: 2026-06-16
decided: 2026-06-16
source: R&D Zone / topic Chat Engine — discovery + keputusan sama pemilik
---

# Chat Engine — Intent Classification Contract Fix

Related: [[chatbot-kasir]], [[2026-06-16-read-intel-intents-roadmap]], [[Chat Engine - Codebase Map]]

## Status
**KEPUTUSAN FINAL (2026-06-16): pakai Opsi B — enum-constrained / structured output.** Scope diciutin ke 1 goal: bikin model tau set intent valid biar gak ngarang intent sendiri. Hanya nyentuh mode 3 (AI teacher). Mode 1 (static) & mode 2 (routing cache) TIDAK disentuh.

## Kenapa note ini ada
Temuan 2026-06-16: **daftar intent gak pernah dikirim ke LLM.** Prasyarat sebelum nambah intent read baru (QUERY_PROFIT dkk) — tanpa fix ini, LLM bisa ngarang intent di luar set & intent baru gak "dikenal" model.

## Konteks arsitektur — routing 3 mode (regeneratif)
Punya pemilik, urutan: **mode 1 static (rule) → mode 2 cek routing cache (DB) → mode 3 AI teacher (LLM)**. Hasil mode 3 dipersist balik ke `routing_examples` / `routing_patterns` / `routing_stats` (chat DB) biar chat mirip berikutnya dilayanin mode 2 tanpa manggil LLM.
- Mode 1 & 2 deterministik → gak kena isu daftar-intent. **Aman, jangan disentuh.**
- Mode 2 cache sengaja **tanpa expiry** (by design, pemilik aware). Bukan bug.
- Prod pakai model kapabel (min haiku / gpt-5.4-mini ke atas), bukan model lokal kecil → isu portability/structured-output-support TIDAK relevan.

## Masalah (akar)
- `OperationalIntent` (Literal 14 nilai) di `app/cognition/operational/schemas.py:9-24` cuma dipakai **validasi Pydantic SETELAH model jawab** — gak pernah ikut ke request.
- Interpreter LLM dipanggil di `_interpret_with_ai` (`app/cognition/operational/service/service.py:312`) → `self.llm_client.generate_response(prompt, route="reasoning")` → balikin **teks mentah** → `_parse_ai_result` (ekstrak JSON manual + `json.loads` + validate).
- `_build_prompt` (`service.py:355-397`) cuma ngirim instruksi + input + contoh schema JSON dengan `"intent"` di-hardcode **`"CREATE_SALE"`** (satu-satunya contoh). Gak ada enforcement output.
- Client: `langchain_openai.ChatOpenAI` (`app/cognition/llm/client.py`), `streaming=True`, model dari env (`LLM_PROVIDER`/`LLM_MODEL`).

## Solusi terpilih — Opsi B: enum-constrained structured output
Daftar intent dipasang sebagai **enum di schema output**, jadi model **secara fisik gak bisa** ngeluarin nilai di luar set (hard guarantee, bukan himbauan). Lawannya Opsi A (prompt-list) cuma "ngasih tau" — ditolak karena B lebih lurus ke goal & model prod udah support.

### Langkah konkret (file per file)

**1. `app/cognition/operational/schemas.py` — sumber kebenaran**
- Enum constraint GRATIS: `InterpretationResult.intent` udah bertipe `OperationalIntent` (Literal). LangChain `with_structured_output` otomatis ngubah Literal → enum di JSON-schema. **Nambah intent ke Literal = otomatis keikut ke enum yang dikirim ke model.** Itu single-source-of-truth-nya.
- Tambah dict `INTENT_CATALOG` (intent → deskripsi 1 baris). BUKAN buat constraint (enum udah handle) — buat bantu model milih intent yang TEPAT. Boleh 1-2 contoh frasa per intent (few-shot mini) buat naikin akurasi.
- Tambah **test guard**: assert tiap anggota `OperationalIntent` punya entry di `INTENT_CATALOG` (nyegah drift kambuh).

**2. `app/cognition/llm/client.py` — jalur structured baru**
- Tambah method, mis. `generate_structured(prompt, schema=InterpretationResult)` → `self._build_model().with_structured_output(InterpretationResult)` → balikin objek tervalidasi.
- Structured = **non-streaming** (gak masalah; klasifikasi gak butuh stream). `generate_response` (streaming) TETAP dipakai buat jawaban akhir ke user. Dua jalur idup bareng: klasifikasi = structured, jawaban = streaming.

**3. `app/cognition/operational/service/service.py` — ganti pemanggilan**
- Di `_interpret_with_ai`: ganti `generate_response` + `_parse_ai_result` → `generate_structured`. `_parse_ai_result` (ekstrak-JSON manual) nyusut/ilang (objek udah valid dari sananya).
- Logika fallback `UNKNOWN` + clarification yang udah ada **DIPERTAHANIN** (jaga-jaga provider error).

**4. `_build_prompt` (service.py) — sederhanain**
- Contoh schema hardcoded `"CREATE_SALE"` udah gak relevan (enum yang nge-handle). Ganti jadi blok teks "Pilihan intent: NAME — deskripsi" di-generate dari `INTENT_CATALOG`. Enum = pagar keras; deskripsi = bantu akurasi.

### Blast radius
3 file (`schemas`, `client`, `service`) + test. Inti: 1 method baru di client, 1 pemanggilan diganti di service, 1 catalog+guard di schemas.

### Wajib disebut ke coder (anti-bingung)
- **Update mock `llm_client` di test existing.** Bentuk pemanggilan berubah (dari "balikin teks" → "balikin objek"). Kalau mock gak diupdate, test 14 intent lama merah karena mock ketinggalan, BUKAN karena logika salah.
- **Regression check**: pastiin 14 intent lama masih ke-detect bener abis perubahan (`_build_prompt` + jalur call kena semua intent, bukan cuma yang baru).

## Catatan
- Fast-path (mode 1) tetep dipertahanin buat intent paling sering (hemat biaya+latency). Buat intent read baru opsional kasih fast-path tipis (QUERY_PROFIT, QUERY_PERIOD_REPORT) — nice-to-have, bukan bagian fix B ini.
- Opsi "katalog dinamis per feature-flag" (exclude CREATE_* yang lagi OFF) = enhancement opsional, bukan blocker goal saat ini.

---

## Refactor terkait — sentralisasi enum (workstream terpisah)

**Goal pemilik:** enum/Literal sekarang nyebar di sana-sini, susah maintain. Pengen disentralisasi ke `app/shared/enums/[enum_name]` biar gampang dirawat.

### Temuan sebaran (verified 2026-06-16) — 8 enum di 8 file
- `app/cognition/operational/schemas.py` → `OperationalIntent`, `DraftType`, `DraftState`
- `app/cognition/routing/__init__.py` → `GraphRoute`, `RouteIntent`, `RouteSource`
- `app/cognition/llm/router.py` → `GraphRoute` ⚠️ **DUPLIKAT** (nama sama, 2 tempat)
- `app/api/schemas/chat.py` → `ChannelProvider` ⚠️ **DUPLIKAT konsep** (nilai identik dgn `ProviderName`)
- `app/domain/channel/entity.py` → `ProviderName` ⚠️ (`web/whatsapp/telegram/instagram/messenger/mobile`)
- `app/domain/operational/entity.py` → `DraftEventType`, `DraftConfirmationType`, `AuditRecordType`
- `app/shared/types.py` → `MessageRole`, `MessageStatus` (**udah di shared** — preseden sentralisasi yg tinggal dilanjutin)

### Yang wajib dikonsolidasi (bukan cuma dipindah)
- `GraphRoute` (2 definisi) → satuin jadi 1 sumber.
- `ChannelProvider` vs `ProviderName` → nilai sama, satuin jadi 1 enum (pilih 1 nama).

### Catatan teknis buat coder
- **Tetep pakai `Literal` type alias, JANGAN dikonversi ke `enum.Enum`.** Pydantic + LangChain `with_structured_output` nge-derive enum JSON-schema dari Literal dengan mulus. Ganti ke `enum.Enum` = perubahan perilaku + risiko ke fix B. Sentralisasi = mindahin lokasi, bukan ganti tipe.
- Ini **refactor import-churn**: low risk logika, tapi nyentuh banyak file (semua importir enum). Wajib **commit/PR terpisah** dari fix B biar diff-nya kebaca.
- `app/shared/types.py` udah ada → boleh jadi rumah, atau pecah ke `app/shared/enums/` sesuai keinginan pemilik. Konsisten satu pola aja.

### Sekuensing vs fix B (rekomendasi)
Idealnya **sentralisasi `OperationalIntent` dluan** (atau barengan), baru fix B numpang di lokasi final — biar `OperationalIntent` + `INTENT_CATALOG` gak kepindah dua kali. TAPI fix B **jangan diblok** sama refactor ini: kalau sentralisasi mundur, B tetep boleh jalan di `schemas.py` dulu, pindah belakangan. Keputusan urutan = pemilik.
