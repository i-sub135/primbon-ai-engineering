---
aliases:
  - "Chat Engine - Deferred Read Intents and Known Constraints"
tags:
  - chatbot-kasir
  - chat-engine
  - intents
  - constraints
  - decision
status: decided-ready-for-handoff
created: 2026-06-18
decided: 2026-06-18
source: R&D Zone / topic Chat Engine — roadmap scoping sama pemilik
---

# Chat Engine — Deferred Read Intents and Known Constraints

Related: [[chatbot-kasir]], [[2026-06-16-read-intel-intents-roadmap]], [[Chat Engine - Codebase Map]]

## Status
**KEPUTUSAN FINAL (18 Jun 2026):** beberapa intent read sengaja ditunda karena fondasi data/logikanya belum cukup matang, dan ada satu constraint penting di sisi debt/payable yang harus dianggap out-of-scope dulu.

## Decision

### Explicitly deferred
Batch sekarang **tidak** mengerjakan:
- `QUERY_EXPENSE_REPORT`
- `QUERY_PROFIT_LOSS`
- `QUERY_STOCK_MOVEMENT`
- `QUERY_CASH_SESSION`
- `QUERY_CUSTOMER`

### Known constraint
`CREATE_DEBT` tidak ikut scope karena ada mismatch domain:
- engine bicara hutang merchant ke supplier
- backend yang terverifikasi sekarang baru jelas punya jalur receivable / debt-sales
- belum ada target tabel `debts` / `payables` yang confirmed

## Rationale
- Expense / profit-loss belum layak karena COA dan posting rules belum matang.
- Stock movement, cash session, dan customer pattern butuh logika analitis tambahan; bukan quick-win batch read-intel ini.
- Memaksa `CREATE_DEBT` masuk sebelum model data backend jelas cuma bikin spec palsu.

## Forward Implication
- Ticketing / coding untuk batch ini jangan menjanjikan Tier 3 intent.
- Kalau salah satu item defer mau dinaikkan prioritas nanti, treat sebagai keputusan baru, bukan implicit carry-over dari roadmap lama.
