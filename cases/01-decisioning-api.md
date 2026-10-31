# Case 01 — One model, one endpoint, forty milliseconds

**Decisioning API · Fintech · 3 weeks · Sole engineer**

> **Anonymized prior work.** Delivered in an earlier role, not as an Aldridge Systems engagement. Client identity, identifying details, exact dates, and commercially sensitive figures are withheld or rounded. The stack and platform behaviour are described as built; the figures are measured results from that engagement, not projections.

## Context

A regulated credit decisioning flow had a validated scorecard, a risk policy in a spreadsheet, and a hard latency budget inside origination. Getting a score still meant opening a notebook, exporting a CSV, and asking the one analyst who knew the column order. The origination service needed a synchronous call: applicant data in, score and decision out, inside a 40 ms p95 budget.

The model had passed model risk review and had to stay. There was no Kubernetes team, no feature store, and no appetite for a six-month platform programme. I had three weeks, one risk analyst as the domain owner, and a constraint that outranked any architecture diagram: the same input and the same model version must produce the same score at the API boundary.

## The problem

A model is not a service. The gap is the work.

The failure modes were operational, not statistical:

- no stable contract between the caller and the model;
- feature order lived in a notebook;
- model version and artifact hash were not observable at the boundary;
- every policy change required a model release;
- rollback meant rebuilding and redeploying.

The brief was not "build less". It was "make the blast radius smaller": one synchronous endpoint, one artifact, one contract, and no state that the origination system already owned.

## Constraints and non-goals

**Constraints**

- p95 under 40 ms at ~1,200 requests/min, measured at the service boundary in `europe-west1`.
- Model version and artifact hash in every health and score response.
- No applicant PII in logs — request IDs and score buckets only.
- Deploy and roll back without dropping a request; one command, no redeploy.
- The risk team must change the decision threshold without a model release.
- Handover to two backend engineers in one week.

**Non-goals**

- No feature store, queue, cache, or database.
- No Kubernetes and no custom autoscaler.
- No in-service retraining; the model is a versioned artifact.
- No public endpoint; the service is private by default.
- No dashboard before the contract tests existed.

## Shape

Five moving parts, and every hand-off explicit:

1. **Caller / identity** — the origination service requests an OIDC identity token and calls the private Cloud Run URL. No API keys in application config; IAM decides who may invoke.
2. **Contract / validation** — Pydantic enforces numeric bounds, a strict request ID, and a 64-applicant batch cap before any tensor is built. Bad input is a 422, never a model error.
3. **Inference / one graph** — one ONNX session per instance, loaded at boot, one intra-op thread, float32 input in a fixed feature order. No DataFrame construction per request.
4. **Envelope / traceability** — the response carries the score, the decision, the model version, the artifact SHA, and the request ID. No features, no names, no raw applicant data.
5. **Platform / rollback** — Cloud Run holds concurrency at 8, keeps one warm instance during business hours, and splits traffic between immutable revisions. Rollback is a traffic command.

A queue would have added a second failure domain and broken the synchronous contract. A cache would have made the score stale and the audit trail ambiguous. A database would have put state inside a decision that is stateless by design. None of them earned their keep.

The risk team owns `DECISION_THRESHOLD`; the platform team owns the image. Policy can change in minutes; the artifact that produces the probability changes only through a tested release.

## Stack

Python 3.12 · FastAPI · Pydantic v2 · ONNX Runtime · Docker · Cloud Run.

One container per instance, one Uvicorn worker, 1 vCPU, 512 MiB, concurrency 8. The ONNX session is created at boot; graph optimisations are applied once; the thread count is pinned to one. The production image does not expose `/docs`, `/redoc`, or `/openapi.json`; the OpenAPI schema is generated in CI and published as an artifact.

## Key decisions

### 01 — Serve ONNX, not a pickled pipeline

**Decision.** Export the fitted pipeline to ONNX, pin the operator set, and serve the graph with ONNX Runtime. The inference image contains neither scikit-learn nor the training stack.

**Why.** The export is a compile step: validate once, hash, test parity. The runtime is small, starts predictably, and has no Python-object deserialisation path on the request boundary.

**Trade-off.** Unsupported operators become release blockers, and an operator change can move the last decimal place. A 5,000-row golden set, batched at the service's 64-applicant cap, gates every image at a maximum absolute difference under `1e-5`.

**What would reopen it.** A model that needs custom preprocessing, or an operator ONNX cannot express. The right move then is to move preprocessing into the contract, not to reintroduce the whole training stack.

### 02 — A strict contract at the boundary

**Decision.** Every numeric field has a plausible range; NaN and infinity are rejected; the request ID is pattern-checked; the batch has a hard cap of 64.

**Why.** A credit score outside the training distribution is not a data-quality nuisance; it is a decision that may be wrong. The contract makes that boundary explicit and returns an actionable 422 before inference — never a 500 or a silent zero.

**Trade-off.** Every new feature requires a schema change and a test. The risk team cannot add a field by changing the caller alone.

**What would reopen it.** A true streaming or variable-length contract would need a different framing. For a synchronous, bounded decision, strict validation is the cheaper failure mode.

### 03 — Cloud Run, one process, concurrency 8

**Decision.** Run one container per instance, one Uvicorn worker, 1 vCPU, 512 MiB, concurrency 8, and one warm instance during business hours. Scale with revisions, not with in-process workers.

**Why.** Cloud Run supplies TLS, identity, revisions, traffic splitting, and scale-to-zero without a cluster. The organisation already had IAM and billing; it did not have a platform team.

**Trade-off.** Cold starts are real (1.9 s), so one instance stays warm during business hours. The service is tied to one cloud's request contract and timeout ceiling. That is the coupling behind "0 proprietary lock-in by design": the artifact is portable, the platform contract is not free. The claim is scoped, not absolute.

**What would reopen it.** GPU inference, multi-cloud residency, or a 10-minute batch workload would outgrow this shape. Those are different products, not a reason to add Kubernetes to this one.

### 04 — Threshold outside the model, owned by risk

**Decision.** The graph returns a probability. `DECISION_THRESHOLD` is an environment value that risk can change without rebuilding the model image.

**Why.** Policy and model quality are different risks with different owners. Separating them lets risk tighten the cutoff during a portfolio review without waiting for an ML release.

**Trade-off.** Threshold and model version must be recorded together. A response that names the model but not the threshold is not auditable.

**What would reopen it.** If policy becomes a multi-variable rule set rather than one cutoff, it should move into a versioned policy service — not grow into a chain of environment variables.

## Measured outcome

| Measure | Result | Scope and limit |
|---|---|---|
| p95 score latency | 38 ms | Against a 40 ms budget at ~1,200 requests/min |
| Cold start | 1.9 s | One warm instance during business hours |
| Compute cost | $0.61 per 1M scores at peak | Compute-only; the warm instance is a separate fixed line |
| Volume | 1.4M scores in the first 90 days | Two model updates; zero P1 incidents |
| Model refresh | Six-week ticket → 25-minute path | Export, parity test, image build, 10% canary, 100% traffic |
| Rollback | Under 90 seconds | One traffic command; the previous revision stays warm |
| Handover | Four-page runbook, contract tests, model card, rollback drill | Two backend engineers now own releases |

The latency number is the least interesting part. The useful result is that a model refresh stopped being a platform project. The risk team changes a threshold; the platform team changes an image; both are visible at the response boundary, and an auditor can tie a decision to an image digest and a policy threshold without reading application logs.

## What would reopen the design

The whole shape reopens if the scorecard becomes a GPU model, if residency requires the same image in another cloud, or if scoring becomes asynchronous. Each changes the contract, not just the runtime. The local triggers are in the decisions above; this is the global one.

## Disclaimer

Anonymized prior work. Delivered in an earlier role, not as an Aldridge Systems engagement. Client identity, identifying details, exact dates, and commercially sensitive figures are withheld or rounded. The stack and platform behaviour are described as built. Nothing here is a guarantee of future availability, latency, or cost.
