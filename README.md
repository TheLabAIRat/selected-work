# Selected work

**Eli Matthew Aldridge · Aldridge Systems**

Three production systems, anonymized and documented. Each one is a decision record first and a result second: the constraint, the trade-off, the measured outcome, and the condition that would reopen the design.

I am an independent MLOps consultant. I take ML and LLM systems the last mile — serving, evaluation, observability, cost control, and the handover nobody scoped. These cases are how I work when the model is already the easy part.

> **Reviewed 31 October 2026.** Claims and anonymization were checked against the landing repository and the owner-confirmed fact base. Keep client permissions and the testimonial release on file.

## What this is

- Three anonymized case studies from earlier roles, written as engineering records rather than marketing.
- A claims-checked set: every figure maps to a source in [CLAIMS.md](CLAIMS.md), with the ambiguous ones named.
- A view of the work that is usually invisible in a portfolio: constraints, non-goals, rejected alternatives, and the conditions that would make a decision wrong.

## What this is not

- **Not a client list.** No client, employer, or product is named. Sector labels are as specific as this draft gets.
- **Not Aldridge Systems engagements.** All three cases were delivered in earlier roles. The disclaimer is repeated on every case.
- **Not a live demo.** There are no endpoints, credentials, or production systems behind these links.
- **Not a guarantee.** Availability and latency figures are historical measurements, not forward-looking promises.
- **Not an absolute portability claim.** The systems are designed for 0 proprietary lock-in, but managed services create real coupling. Case 01 names its Cloud Run dependency rather than hiding it.
- **Not a pitch deck.** There is no urgency, no countdown, and no "book a call" button. The work is the argument.

## At a glance

| | |
|---|---|
| Experience | 2+ years in production ML and MLOps |
| Systems shipped | 6+ production systems |
| Availability | 99.95% median after handover — a historical median, not a guarantee |
| Delivery | 10 weeks median prototype-to-production |
| Lock-in | 0 proprietary lock-in by design; managed-service coupling is named where it exists |
| Sectors | IT, logistics, fintech, healthcare |
| Capacity | 1 engagement at a time |

The three cases below cover fintech, logistics, and healthcare. IT is part of the wider sector list but is not represented in this selection.

## Selected work

### [01 — One model, one endpoint, forty milliseconds](cases/01-decisioning-api.md)

**Fintech · decisioning API · 3 weeks · sole engineer**

A validated credit scorecard was trapped behind a notebook and a CSV export, inside a 40 ms p95 budget. The answer was one container, one contract, and no cache, queue, or database. **Result:** 38 ms p95, 1.4M scores in 90 days, zero P1 incidents.

### [02 — The forecast was right. Monday disagreed.](cases/02-demand-forecasting.md)

**Logistics · demand forecasting · 10 weeks · lead MLOps engineer**

The model beat its incumbent in backtests while planners overrode 38% of its recommendations. The bug was not the model; the training join used warehouse ingestion time instead of event time. **Result:** WAPE 24.6% → 17.0%, excess inventory −22%, freshness p95 12 minutes against a 15-minute SLO.

### [03 — An LLM gateway that can show its work](cases/03-governed-llm-gateway.md)

**Healthcare · governed LLM gateway · 16 weeks · lead platform engineer**

Six teams, three providers, and no way to prove why a clinical summary was allowed to leave the building. The gateway put identity, redaction, policy, and audit on one request path. **Result:** 47% lower cost per 1,000 requests, 18 ms p95 gateway overhead, and an audit record on 100% of policy decisions.

## How to read

Each case follows the same order:

1. **Context** — the system, the constraint, and what was deliberately out of scope.
2. **The problem** — the failure mode, stated without the solution.
3. **Shape** — the request or data path, in the smallest number of moving parts.
4. **Stack** — what was actually used, not a logo wall.
5. **Key decisions** — three or four choices, each with the alternative, the trade-off, and what would reopen it.
6. **Measured outcome** — the numbers, their scope, and their limits.
7. **What would reopen the design** — the global condition, not just the local one.
8. **Disclaimer** — the anonymization and provenance note.

If you read one case, read the decisions. The outcome is the receipt; the decisions are the work. If a number is not in [CLAIMS.md](CLAIMS.md), treat it as missing rather than approximate.

## Engagements

| Engagement | Shape | From |
|---|---|---|
| Autonomy Audit | 2 weeks | $12k |
| Production Sprint | 4–6 weeks | $36k |
| Fractional MLOps | Fixed monthly scope | $9.5k / month |

One engagement at a time.

## Contact

[GitHub](https://github.com/TheLabAIRat) · [X](https://x.com/TheLabAIRat)

The portfolio site is intentionally not linked from this draft while it is hidden.

---

*Prepared 31 October 2026. Source of truth: the landing repository's case pages, README, content checklist, and owner-confirmed fact base. See [CLAIMS.md](CLAIMS.md) for the ledger and the checks run.*
