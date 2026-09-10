---
aliases:
  - "Chatbot Kasir - ADR-017: Skema Log LLM Global, Titik Emisi Tunggal"
---

# Chatbot Kasir - ADR-017: Skema Log LLM Global, Titik Emisi Tunggal

**Status:** Accepted — 2026-09 W36 (V2.FTR-13/14/15), landasan ENC-13/14 (structlog JSON, W34)

## Context

Worker cognition nol visibility pemakaian LLM (token, latency, error). Log mau jadi sumber data Grafana/Loki — butuh skema stabil sebelum dashboard dibuat.

## Decision

- Satu util global `app/shared/logging/llm_event.py` = satu-satunya pembentuk payload event `llm.chat_completion` / `llm.route_classification`.
- Skema: **metric flat di root** (aman jadi label Loki: `usage_*`, `duration_ms`, `provider`, `tenant_id`, `store_id`, `status`, `intent`, `route_*`) + **teks manusia nested di `content`** (haram jadi label; cap 500 char, `*_chars` simpan panjang asli; bisa dimatiin via env `LLM_LOG_CONTENT`).
- Titik emisi TETAP satu di `client.py` — bukan per call site. Fallback path & error path emit baris sendiri (sebelumnya bisu).
- Breaking rename diambil sekarang (`input_tokens`→`usage_input` dst) karena konsumen log belum ada.

## Rationale

Coverage bolong = panel Grafana ngasih angka "keliatan valid tapi salah" tanpa tanda error — lebih bahaya dari gak ada panel. Emisi tunggal bikin bolong mustahil secara struktur.

## Alternatives Considered

Log per call site (nolak: coverage bolong senyap), nunggu dashboard dulu baru rename (nolak: bayar migrasi).

## Forward Implication

Residual sadar-diambil: `duration_ms` = attempt terakhir (retry rate invisible); error yang ditelan layer SDK gak pernah nyampe `except` → error rate bisa 0 palsu pas insiden provider. `res_truncated` diukur setelah strip markdown, `res_chars` sebelum — panel yang ngandelin `*_chars` baca catatan FTR-14.

## Implementation Pointer

`app/shared/logging/llm_event.py`, emisi di `app/cognition/llm/client.py`, state pass-through di `app/cognition/graph/nodes.py`. Ticket: V2.FTR-13, V2.FTR-14, V2.FTR-15 (task-archive).
