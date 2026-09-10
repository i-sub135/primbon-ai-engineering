---
aliases:
  - "Chatbot Kasir - ADR-015: gpt-5.6-luna Sebagai Model Intent + Chat"
---

# Chatbot Kasir - ADR-015: `gpt-5.6-luna` Sebagai Model Intent + Chat

**Status:** Accepted — 2026-08-13 (build staging `13.8.26-2`)

## Context

`deepseek-v4-pro` akurat tapi lambat (9–41s per intent) — UX chat gak layak, timeout risk tinggi.

## Decision

Ganti model intent classification + chat ke `gpt-5.6-luna`. Probe regresi penuh sebelum ketok: 28 varian query, 8 intent, staging.

## Rationale

28/28 PASS (zero regresi), latency turun 3–14x (avg ~2.94s; QUERY_TOP_PRODUCTS ~41s → ~3.4s), route 100% dispatcher tanpa fallback graph. Report: `docs/reports/2026-08-intent-probe-gpt-5.6-luna.md`.

## Alternatives Considered

Tetap `deepseek-v4-pro` (nolak: latency), `glm-5.2` (dipakai sebagai model lain di gateway opencode.ai; punya quirk reasoning_effort sendiri).

## Forward Implication

Model ini punya dua jebakan yang udah dibayar mahal: `reasoning_effort` wajib `none` (V2.BUG-17) dan `stream_usage=True` wajib eksplisit (V2.BUG-18). Ganti model lagi = ulangi probe 28-varian + daftarin quirk barunya. Cost per-token belum pernah dibandingin (di luar scope probe).

## Implementation Pointer

`app/cognition/llm/client.py` (`_OPENAI_REASONING_EFFORT_BY_MODEL`, `_build_model`), env `LLM_API_BASE_URL`.
