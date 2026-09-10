---
aliases:
  - "Chatbot Kasir - ADR-016: Daily Chat Quota Per User"
---

# Chatbot Kasir - ADR-016: Daily Chat Quota Per User (Cost Control)

**Status:** Accepted — 2026-08 W34 (V2.ENC-11, refined V2.BUG-15)

## Context

Token LLM unbounded per user — estimasi cost-bleed puluhan juta/bulan (~Rp250/chat) kalau dibiarin saat merchant nambah.

## Decision

Quota harian **25 chat/user** (env `CHAT_DAILY_LIMIT`), window calendar-day WIB. Dua fase: display (`chat_count_today` + `chat_limit` di history endpoint) lalu enforce (HTTP 429 `{"error": "error limit"}` di `POST /chat`). Terpisah dari slowapi per-menit limiter. Row `status='history'` (echo log FE) di-exclude dari hitungan.

## Rationale

Cost predictable per user per hari; 429 eksplisit lebih jujur ke FE daripada silent throttle; WIB window konsisten sama ADR-011.

## Alternatives Considered

Token-based budget (nolak: susah dikomunikasiin ke merchant), rate limit per menit doang (nolak: gak nutup volume harian).

## Forward Implication

Angka 25 = knob bisnis, bukan konstanta teknis — naikin via env tanpa deploy. Query quota WAJIB filter status (Pitfall 34).

## Implementation Pointer

`count_user_chats_today()` (shared query), gate di `POST /chat`, env `CHAT_DAILY_LIMIT` (`.env.example` per 2026-09-03).
