---
aliases:
  - "Chat Engine - Read-Intel Audit Execution Sequencing"
tags:
  - chatbot-kasir
  - chat-engine
  - audit
  - sequencing
  - decision
status: decided-ready-for-handoff
created: 2026-06-18
decided: 2026-06-18
source: R&D Zone / topic Chat Engine — prioritisasi sama pemilik
---

# Chat Engine — Read-Intel × Audit Execution Sequencing

Related: [[chatbot-kasir]], [[2026-06-16-read-intel-intents-roadmap]], [[2026-06-16-intent-classification-contract-fix]], [[audit-chat-engine-quality-maturity]], [[Chat Engine - Codebase Map]]

## Status
**KEPUTUSAN FINAL (18 Jun 2026):** urutan eksekusi read-intel mengikuti sequencing fondasi-audit dulu, baru fitur, supaya workstream audit dan roadmap tidak saling tumpang tindih.

## Decision

### Fase 0 — fondasi audit ∩ roadmap
1. `ARCH-4` / sentralisasi enum via [[2026-06-16-intent-classification-contract-fix]]
2. `DATA-2` tenant scoping jalur report
3. `DATA-1` soft-delete guard di jalur produk
4. Shared helper ter-test untuk 5 aturan global query read

### Fase 1 — resilience & guardrail
5. `ERR-1` timeout/retry untuk LLM call
6. `ERR-3` ack-on-success / dead-letter safety
7. `pytest-cov` + CI gate minimal

### Fase 2 — feature delivery
8. `QUERY_PROFIT`
9. `QUERY_PERIOD_REPORT`
10. `QUERY_RECEIVABLES`
11. Tier 2 sesudah Tier 1 stabil

## Rationale
- Sebagian finding audit itu prasyarat langsung buat roadmap read-intel, bukan lane terpisah.
- `DATA-2` harus selesai sebelum `QUERY_PERIOD_REPORT` karena sama-sama bertumpu di `daily_sales_summaries`.
- `DATA-1` harus selesai sebelum query berbasis produk/kategori supaya bug soft-delete tidak menyebar ke intent baru.
- Hardening `ERR-1` dan `ERR-3` menurunkan risiko saat jumlah intent read bertambah.

## Guardrails
Lima aturan global tetap dianggap wajib di semua intent read:
1. tenant scoping
2. soft delete
3. exclude transaksi batal
4. timezone merchant
5. mirror precedent intent read existing

## Forward Implication
- Ticket hasil QA dibaca dengan dependency per fase, bukan sebagai backlog datar.
- Detail argumentasi audit-roadmap lama tidak perlu dipertahankan di `work-ready`; decision ini jadi source of truth sequencing.
