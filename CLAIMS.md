# Claims ledger

Every number and factual claim in this repository maps to a source below. If a claim is not in this ledger, it is not a claim; treat it as missing rather than approximate.

**Source key.** Paths such as `cases/simple.html`, `README.md`, `CONTENT-CHECKLIST.md`, and `x-content/fact-base.md` refer to the landing repository (the read-only source of truth), not to files in this repository. "Owner-confirmed" means the figure was confirmed by the site owner on 31 October 2026 and recorded in `CONTENT-CHECKLIST.md` or `x-content/fact-base.md`. "Case page" means the figure is stated in the corresponding anonymized case HTML. "Derived" means arithmetic from a sourced figure, with the arithmetic shown.

> Line references were captured while the landing pages were being edited and may drift. Treat the claim wording plus the case section title as the stable anchor; re-run the line check before publication. The status column is the binding part.

## Practice claims (README)

| Claim | Wording in this repository | Source | Status |
|---|---|---|---|
| Name | Eli Matthew Aldridge | `x-content/fact-base.md:9` | Owner-confirmed |
| Practice | Aldridge Systems, independent MLOps consultancy | `x-content/fact-base.md:10` | Owner-confirmed |
| Experience | 2+ years in production ML and MLOps | `CONTENT-CHECKLIST.md:12`; `x-content/fact-base.md:12` | Owner-confirmed |
| Sectors | IT, logistics, fintech, healthcare | `CONTENT-CHECKLIST.md:12`; `x-content/fact-base.md:13` | Owner-confirmed |
| Systems shipped | 6+ production systems | `CONTENT-CHECKLIST.md:34`; `x-content/fact-base.md:14` | Owner-confirmed |
| Availability | 99.95% median after handover; historical median, not a guarantee | `CONTENT-CHECKLIST.md:35`; `x-content/fact-base.md:15`; caveat `x-content/fact-base.md:98` | Owner-confirmed |
| Delivery | 10 weeks median prototype-to-production | `CONTENT-CHECKLIST.md:36`; `x-content/fact-base.md:16` | Owner-confirmed |
| Lock-in | 0 proprietary lock-in by design; managed-service coupling named where it exists | `CONTENT-CHECKLIST.md:37`; `x-content/fact-base.md:17`; Cloud Run coupling `cases/simple.html:565` | Owner-confirmed, scoped |
| Capacity | 1 engagement at a time | `CONTENT-CHECKLIST.md:13`; `x-content/fact-base.md:18` | Owner-confirmed |
| Autonomy Audit | 2 weeks, from $12k | `CONTENT-CHECKLIST.md:47` | Owner-confirmed |
| Production Sprint | 4–6 weeks, from $36k | `CONTENT-CHECKLIST.md:48` | Owner-confirmed |
| Fractional MLOps | Fixed monthly scope, from $9.5k / month | `CONTENT-CHECKLIST.md:49` | Owner-confirmed |
| Cases are prior work | All three cases were delivered in earlier roles; not Aldridge Systems engagements | `CONTENT-CHECKLIST.md:56-57`; `x-content/fact-base.md:25`; `README.md:124-127`; `README.md:368-372` | Source |
| No client names | No client, employer, or product is named; sector labels only | `README.md:124-127`; `x-content/fact-base.md:94` | Source |
| Contact: GitHub | `https://github.com/TheLabAIRat` | `CONTENT-CHECKLIST.md:22`; HTTP 200 on 31 October 2026 | Verified |
| Contact: X | `https://x.com/TheLabAIRat` | `CONTENT-CHECKLIST.md:23`; HTTP 200 on 31 October 2026 | Verified |
| Portfolio site omitted | Site intentionally not linked while hidden | `README.md:217-232` | Source; no URL used |
| Draft date | Prepared 31 October 2026 | `CONTENT-CHECKLIST.md:3`; `x-content/fact-base.md:7`; `README.md:368` | Source |
| No live endpoints | No endpoints, credentials, or production systems in this repository | `cases/simple.html:620`; `cases/medium.html:623`; `cases/complex.html:740`; repository contents | Structural |

## Case 01 — Decisioning API

| Claim | Wording in this repository | Source | Status |
|---|---|---|---|
| Sector and domain | Fintech; decisioning API | `cases/simple.html:64` | Case page |
| Timeline | 3 weeks, one release candidate | `cases/simple.html:77` | Case page |
| Role | Sole engineer — export, service, delivery, handover | `cases/simple.html:76` | Case page |
| Team | 1 engineer + 1 risk analyst | `cases/simple.html:78` | Case page |
| Peak load | ~1,200 requests/min | `cases/simple.html:79`, `:142` | Case page |
| Latency budget | 40 ms p95 at the service boundary in europe-west1 | `cases/simple.html:88`, `:142` | Case page |
| Measured latency | 38 ms p95 | `cases/simple.html:87`, `:773` | Case page |
| Cold start | 1.9 s | `cases/simple.html:91` | Case page |
| Warm instance | One warm instance during business hours | `cases/simple.html:200`, `:565` | Case page |
| Compute cost | $0.61 per 1M scores at peak; compute-only, warm instance separate | `cases/simple.html:95`, `:777` | Case page |
| Volume | 1.4M scores in the first 90 days | `cases/simple.html:773` | Case page |
| Model updates | Two | `cases/simple.html:773` | Case page |
| Incidents | Zero P1 | `cases/simple.html:773` | Case page |
| Golden set | 5,000 real rows, 64-applicant cap, max absolute difference under 1e-5 | `cases/simple.html:301` | Case page |
| Runtime shape | One container, one Uvicorn worker, 1 vCPU, 512 MiB, concurrency 8 | `cases/simple.html:565` | Case page |
| Inference setup | ONNX session at boot, graph optimisations once, one intra-op thread, fixed feature order | `x-content/fact-base.md:37`; `cases/simple.html:250`, `:260` | Case page / owner-confirmed |
| No interactive docs | Production image does not expose /docs, /redoc, or /openapi.json | `x-content/fact-base.md:39`; `cases/simple.html:255` | Case page / owner-confirmed |
| Traceability | Model version and artifact hash in every health and score response | `cases/simple.html:143`, `:195`, `:265` | Case page |
| No applicant PII in logs | Request IDs and score buckets only | `cases/simple.html:144` | Case page |
| Invalid input | 422 before inference | `cases/simple.html:185` | Case page |
| Batch cap | 64 applicants | `cases/simple.html:185` | Case page |
| Private endpoint | No public endpoint; private by default | `cases/simple.html:156` | Case page |
| Model refresh | Six-week ticket → 25-minute path: export, parity test, image build, 10% canary, 100% traffic | `cases/simple.html:771` | Case page |
| Rollback | Under 90 seconds; one traffic command; previous revision stays warm | `cases/simple.html:772` | Case page |
| Handover | Four-page runbook, contract tests, model card, rollback drill; two backend engineers own releases | `cases/simple.html:774` | Case page |
| No state | No cache, queue, or database | `cases/simple.html:597` | Case page |
| Cloud Run coupling | Service tied to one cloud's request contract and timeout ceiling | `cases/simple.html:565`; `x-content/fact-base.md:17` | Case page / owner-confirmed |

## Case 02 — Demand forecasting

| Claim | Wording in this repository | Source | Status |
|---|---|---|---|
| Sector and domain | Logistics; demand forecasting | `cases/medium.html:64` | Case page |
| Timeline | 10 weeks, four regions | `cases/medium.html:77` | Case page |
| Role | Lead MLOps engineer — contracts, orchestration, monitoring, handover | `cases/medium.html:76` | Case page |
| Scale | 1,800 SKUs × 4 regions | `cases/medium.html:79` | Case page |
| Cadence | Daily batch | `cases/medium.html:79` | Case page |
| Freshness SLO | 15-minute source-to-publish | `cases/medium.html:79`, `:96` | Case page |
| Backtest comparison | Beat the incumbent by 18% in backtests | `cases/medium.html:135` | Case page |
| Override rate | Planners overrode 38% of recommendations | `cases/medium.html:135` | Case page |
| Root causes | Ingestion time used as event time; promotion calendar after feature build; carrier feed stale 6–30 hours; no drift visibility; notebook retraining | `cases/medium.html:142-146` | Case page |
| TTL | 36 hours | `cases/medium.html:249` | Case page |
| Band check | 25% median change blocks publish | `cases/medium.html:568` | Case page |
| Drift method | PSI on every column; dataset drift above a 30% share | `cases/medium.html:315`, `:587` | Case page |
| Shadow rollout | Checks ran in shadow mode for two weeks; override records a reason and an approver | `cases/medium.html:571` | Case page |
| Retraining gate | Weekly; eight-week backtest; seven-day shadow; WAPE improvement at least 2%; bias within ±3% | `cases/medium.html:606` | Case page |
| WAPE | 24.6% → 17.0% | `cases/medium.html:87-88`, `:772` | Case page |
| Relative reduction | 31% | `cases/medium.html:88`; derived: (24.6 − 17.0) / 24.6 = 30.9%, rounded to 31% | Case page / derived |
| Excess inventory | −22% across four regions; stockouts stayed flat | `cases/medium.html:91`, `:773` | Case page |
| Freshness | 12 minutes p95 against a 15-minute SLO | `cases/medium.html:95`, `:96` | Case page |
| Overrides | 38% → 22%; every override carries a reason code | `cases/medium.html:774` | Case page |
| Detection | Regional carrier change caught in 36 hours, not five weeks | `cases/medium.html:775` | Case page |
| Stack | Python, Dagster, Feast, Evidently, Prometheus, Grafana, Postgres, Parquet/S3 | `cases/medium.html:81` | Case page |
| Feature contract | replenishment_v3 | `cases/medium.html:264`, `:375` | Case page |
| Model churn | Model changed once; feature path changed completely | `cases/medium.html:772` | Case page |

## Case 03 — Governed LLM gateway

| Claim | Wording in this repository | Source | Status |
|---|---|---|---|
| Sector and domain | Healthcare; governed LLM gateway | `cases/complex.html:64` | Case page |
| Timeline and scale | 16 weeks, six teams, three providers | `cases/complex.html:77` | Case page |
| Model and volume | 9 models, ~40M requests/month | `cases/complex.html:79` | Case page |
| Cost | 47% lower cost per 1,000 requests | `cases/complex.html:87`, `:935` | Case page |
| Gateway overhead | 18 ms p95 against a 25 ms budget; alerting at 22 ms | `cases/complex.html:91-92`, `:936` | Case page |
| Audit coverage | 100% of policy decisions with a decision ID, trace ID, and policy version; redaction labels recorded | `cases/complex.html:95-96`, `:937` | Case page |
| Overhead split | 4 ms redaction, 5 ms OPA; rest in routing and serialization | `cases/complex.html:212` | Case page |
| Eval suite | 240 clinical cases | `cases/complex.html:208`, `:685` | Case page |
| Canary | 5% → 25% → 100% via Argo Rollouts | `cases/complex.html:208`, `:685` | Case page |
| Audit retention | Seven years | `cases/complex.html:147` | Case page |
| Providers | Azure OpenAI, Anthropic, self-hosted vLLM | `cases/complex.html:155` | Case page |
| Identity | Virtual key resolves to a team, model allowlist, and budget; auth middleware writes team and data class | `cases/complex.html:183`, `:221`, `:292` | Case page |
| Redaction order | Redaction runs before policy; failed redaction returns 422 without a provider call | `cases/complex.html:212`, `:717`; `x-content/fact-base.md:69` | Case page / owner-confirmed |
| Policy input | OPA receives the redacted prompt, prompt length, and redaction labels — not raw clinical text | `cases/complex.html:193`, `:212`; `x-content/fact-base.md:69` | Case page / owner-confirmed |
| Redaction detectors | SSNs, MRNs, emails, and phone numbers replaced with placeholders; reversible map stays inside the boundary | `cases/complex.html:188` | Case page |
| Semantic PHI | Separate classifier with its own fail-closed path | `cases/complex.html:726` | Case page |
| Eval maintenance | Weekly failure review; adversarial PHI cases added monthly; reviewed exception path for hotfixes | `cases/complex.html:688` | Case page |
| Policy result | Allow plus policy version; decision ID created by the gateway, passed into policy input, recorded with the trace | `cases/complex.html:193`, `:320`; `x-content/fact-base.md:69` | Case page / owner-confirmed |
| No raw PHI in decision logs | Policy decision log contains redacted input and labels, not raw clinical text | `cases/complex.html:725`, `:937`; `x-content/fact-base.md:69` | Case page / owner-confirmed |
| Budgets | Per-team keys, quotas, hard monthly budget; 402 at 100% with a reset date; no silent cheaper-model fallback | `cases/complex.html:143`; `x-content/fact-base.md:71` | Case page / owner-confirmed |
| Observability | OpenTelemetry GenAI spans; Langfuse redacted traces; ClickHouse audit record and cost ledger | `cases/complex.html:203` | Case page |
| Routing layer | LiteLLM for model groups, fallbacks, timeouts, retries, Redis cooldown | `cases/complex.html:81`, `:628` | Case page |
| Deep tier | Self-hosted vLLM for the deep tier | `cases/complex.html:155`, `:659-677` | Case page |
| Gateway resilience | Three replicas across zones; degraded mode for provider outages | `cases/complex.html:612` | Case page |
| Policy resilience | Extra hop 5 ms p95; policy outage fails closed; sidecar replicated; bundle cached locally | `cases/complex.html:650` | Case page |
| Eval gate | Deterministic, auditable; reviewed exception path for hotfixes | `cases/complex.html:687` | Case page |

## Omitted or ambiguous claims

These were considered and deliberately not used. They are recorded here so a reviewer can see the boundary of the draft.

| Claim or detail | Source | Why it is not used |
|---|---|---|
| Zero confirmed PHI egress incidents in six months | `cases/complex.html:938` | Present on the case page but not in the owner-confirmed fact list. A safety result with this scope needs owner sign-off before publication. |
| Two attempted disallowed-model calls denied before egress and reviewed the same day | `cases/complex.html:938` | Same reason as above; omitted with the PHI-egress result. |
| The source's client descriptor for Case 03 | `cases/complex.html:135` | The fact base explicitly says not to use it (`x-content/fact-base.md:78`). The draft uses "regulated healthcare platform" and the sector label. |
| The source's client descriptor for Case 02 | `cases/medium.html:135` | The fact base says not to use the sector-specific client descriptor or exact regions (`x-content/fact-base.md:59`). The draft uses "multi-region replenishment operation". |
| The source's client descriptor for Case 01 | `cases/simple.html:66` | The fact base explicitly says not to use it (`x-content/fact-base.md:43`). The draft uses "regulated credit decisioning flow". |
| "Most requested" service flag | `CONTENT-CHECKLIST.md:50` | Checklist says keep only if true; not confirmed. |
| Testimonial and attribution | `CONTENT-CHECKLIST.md:66-68` | Requires a real quote and release; out of scope for this draft. |
| Funding stage or investor claim | No source | No funding claim is made. |
| Absolute portability claim | `x-content/fact-base.md:99` | The fact base forbids an absolute; the draft uses the scoped "0 proprietary lock-in by design" and names the Cloud Run coupling. |
| Claim that OPA receives raw clinical text | `x-content/fact-base.md:78`; `cases/complex.html:717` | Contradicts the architecture. The draft states the opposite: redaction runs first, and OPA receives redacted input plus labels. |
| Exact dates, regions, or client-identifying details | `x-content/fact-base.md:25` | Withheld by design. |
| "Six-week ticket" and "25-minute path" | `cases/simple.html:771` | Retained; both are explicit case-page results. |

## Link verification

Checked on 31 October 2026 with `curl -sS -o /dev/null -w "%{http_code}" -L`.

| Link | Used in | Result |
|---|---|---|
| https://github.com/TheLabAIRat | README, Contact | 200 |
| https://x.com/TheLabAIRat | README, Contact | 200 |
| `cases/01-decisioning-api.md` | README, case card 01 | Resolves locally |
| `cases/02-demand-forecasting.md` | README, case card 02 | Resolves locally |
| `cases/03-governed-llm-gateway.md` | README, case card 03 | Resolves locally |
| `CLAIMS.md` | README, two references | Resolves locally |

No other external URLs are used. The hidden portfolio site is not linked.

## Consistency checks

Repository-wide checks run before commit. The literal patterns are held in the review transcript rather than repeated here, so this file does not reintroduce the wording it is checking for.

| Check | Result |
|---|---|
| No stray year from an earlier era | Pass — no pre-2020 date appears |
| No abbreviated week unit | Pass — week durations are written out |
| No provider-count mismatch | Pass — three providers, consistent with the source |
| No placeholder token | Pass — no placeholder marker |
| No absolute portability wording | Pass — scoped claim only; Cloud Run coupling named |
| No claim that OPA receives raw clinical text | Pass — redaction-before-policy is stated consistently |
| No client, employer, or product names | Pass — sector labels and anonymized descriptors only |
| No funding-stage claim | Pass — none present |
| No tenure inflation | Pass — the only tenure claim is 2+ years |
| Case 03 architecture consistency | Pass — auth middleware, redaction before policy, redacted OPA input, allow plus policy version, decision ID |
| "99.95%" only as a median | Pass — historical median, not a guarantee |
| "$0.61" only as compute-only at peak | Pass — warm instance called out as a separate fixed line |
| "10 weeks" used consistently | Pass — practice median and Case 02 timeline, both sourced |
| "4–6 weeks" only in pricing | Pass — Production Sprint only |
| Every internal link resolves | Pass — all four targets exist |

## Known uncertainty for the reviewer

- The case figures are stated in the case pages; they have not been independently re-audited for this draft. The owner should confirm they remain accurate and non-identifying.
- The 99.95% availability figure is a historical median after handover, not a forward-looking guarantee. The same applies to latency and cost figures.
- "0 proprietary lock-in by design" is a design statement, not an absolute. Case 01 names the Cloud Run coupling explicitly.
- The role labels "Lead MLOps engineer" and "Lead platform engineer" come from the case pages. Confirm they are the titles the owner wants to publish.
- The 31% relative WAPE reduction is both stated in the source and reproducible from 24.6% and 17.0%; the arithmetic rounds to 30.9%.
- The 47% cost reduction is a point estimate from the case page. No baseline dollar figure is published, so the percentage cannot be independently recomputed from this repository.
- Client permissions and a testimonial release are not on file in this repository. Publication requires them.
- Pricing is owner-confirmed as of 31 October 2026 and should be re-confirmed before publication.
- The GitHub and X links returned 200 on 31 October 2026; they can change.
