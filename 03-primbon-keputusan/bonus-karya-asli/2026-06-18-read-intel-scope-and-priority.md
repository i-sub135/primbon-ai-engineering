---
aliases:
  - "Chat Engine - Read-Intel Scope and Priority"
tags:
  - chatbot-kasir
  - chat-engine
  - intents
  - reporting
  - decision
status: decided-ready-for-handoff
created: 2026-06-18
decided: 2026-06-18
source: R&D Zone / topic Chat Engine — discovery 16 Jun + formalisasi keputusan 18 Jun
---

# Chat Engine — Read-Intel Scope and Priority

Related: [[chatbot-kasir]], [[2026-06-16-read-intel-intents-roadmap]], [[2026-06-16-intent-classification-contract-fix]], [[Chat Engine - Codebase Map]]

## Status
**KEPUTUSAN FINAL (18 Jun 2026):** pengembangan read-intelligence jalan dengan scope **READ-first**, sementara semua intent `CREATE_*` tetap di-skip selama `feature_transactional_enabled=OFF`.

## Decision

### 1. Scope delivery
- Fokus delivery sekarang = intent baca / reporting / query.
- Intent write/transaksional `CREATE_*` tidak ikut batch ini.
- [[2026-06-16-intent-classification-contract-fix]] tetap jadi prasyarat routing sebelum nambah intent read baru.

### 2. Priority order untuk Tier 1
Urutan prioritas yang dipakai:
1. `QUERY_PROFIT`
2. `QUERY_PERIOD_REPORT`
3. `QUERY_RECEIVABLES`

Tier 2 (`QUERY_PAYMENT_BREAKDOWN`, `QUERY_TOP_PRODUCTS`, `QUERY_SALES_BY_CATEGORY`) nyusul setelah Tier 1 stabil.

## Rationale
- Nilai bisnis paling cepat kerasa ada di sisi read; engine existing timpang karena write sudah jauh lebih lengkap dari read.
- `QUERY_PROFIT` paling tinggi leverage-nya buat merchant karena sekarang engine baru sampai omzet, belum laba/margin.
- `QUERY_PERIOD_REPORT` murah dibangun setelah fondasi benar karena bisa numpang `daily_sales_summaries`.
- `QUERY_RECEIVABLES` jadi jembatan ke alert/proactive follow-up tanpa harus nyentuh write-path.

## Forward Implication
- Ticket, QA, dan handoff coder untuk batch read-intel wajib ngikut scope ini.
- Detail spec per intent tetap hidup di [[2026-06-16-read-intel-intents-roadmap]] sebagai reference, bukan decision.
