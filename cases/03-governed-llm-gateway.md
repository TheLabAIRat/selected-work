# Case 03 — An LLM gateway that can show its work

**Governed LLM gateway · Healthcare · 16 weeks · Lead platform engineer**

> **Anonymized prior work.** Delivered in an earlier role, not as an Aldridge Systems engagement. Client identity, identifying details, exact dates, and commercially sensitive figures are withheld or rounded. The stack and policy model are described as built; the figures are measured results from that engagement, not projections.

## Context

A regulated healthcare platform grew from one chatbot to six product teams calling three providers directly from application code. Each team had its own SDK, retry logic, prompt logs, and budget spreadsheet. A security review found three problems that no team could solve alone: PHI could reach a provider without redaction; per-team spend was invisible until the invoice; and no one could reconstruct why a model produced a clinical summary — which prompt, which model, which policy, which version.

The platform did not need another SDK. It needed a place where those questions had one answer.

## The problem

Six teams across three providers meant six upgrade paths, six policy implementations, and no single audit record. The gateway would become a third-party dependency on every request, so the design problem was keeping that dependency small, observable, and reversible.

The hard part was not routing tokens. It was proving who asked what, which model answered, and why the answer was allowed to leave the building — without putting raw clinical text into the audit trail.

## Constraints and non-goals

**Constraints**

- One OpenAI-compatible endpoint; existing applications keep their SDKs.
- Per-team keys, quotas, and a hard monthly budget with a visible reset date.
- Policy on every request, evaluated before egress, with a decision ID.
- PHI redaction before any provider call; fail closed if confidence is low.
- Policy input and decision logs contain redacted text and labels, never raw PHI.
- Prompt and response traces with retention controls; audit records for seven years.
- Eval gates before a model or prompt promotion; canary and rollback.
- p95 gateway overhead under 25 ms.
- Three providers: Azure OpenAI, Anthropic, and self-hosted vLLM.
- Existing Kubernetes; no new control plane.
- Security owns the policy; the platform owns the runtime.
- A provider outage must degrade the request path, not fail it.

**Non-goals**

- A shared SDK that each team imports. That keeps six upgrade paths, six policy implementations, and no single audit record.
- Policy as Python functions inside the gateway. Every rule change becomes an application deploy, and the audit trail is whatever the code chooses to log.
- Sending raw clinical text to OPA so policy can inspect it. Privacy wins over a few milliseconds on denied traffic.
- A second uncontrolled provider path. A provider feature the gateway cannot express gets a supported escape hatch with its own audit.

## Shape

Six steps, one audit trail. Everything that can happen after the response — tracing, cost aggregation, eval sampling — is moved off the critical path.

1. **Identity / team key** — a virtual key resolves to a team, a model allowlist, and a budget. Internal services use OIDC. The auth middleware writes `request.state.team` and `request.state.data_class`; the gateway never trusts a team name supplied by the caller.
2. **Redaction / fail closed** — deterministic detectors replace SSNs, MRNs, emails, and phone numbers with placeholders before policy runs. The reversible map stays inside the boundary; if redaction cannot complete, the request is denied with a 422 and no provider call.
3. **Policy / OPA** — OPA evaluates team, model, data class, prompt size, and redaction labels. The input is already redacted. The policy returns allow and the policy version. The decision carries the verdict, the policy version, and the decision ID that the gateway created before the call and passed in as input.
4. **Routing / fallback** — LiteLLM routes by model group, applies timeouts and one retry, and falls back from the hosted fast tier to the self-hosted deep tier. Redis holds cross-worker cooldown and spend state.
5. **Trace / audit** — OpenTelemetry GenAI spans carry model, token usage, latency, and trace ID. Langfuse stores redacted prompt and response traces; ClickHouse holds the audit record and cost ledger.
6. **Release / eval gate** — a 240-case clinical suite runs before promotion. Argo Rollouts moves traffic 5% → 25% → 100%; a failed quality, latency, or cost guardrail rolls back automatically.

The gateway spends 4 ms in redaction, 5 ms in OPA, and the rest in routing and serialization. Redaction runs before policy so the decision log never becomes a PHI store. That budget is why policy cannot be a Python function that grows a new `if` every sprint: it has to be a versioned artifact, testable outside the request path, with a decision log the security team can query.

Budgets are enforced before the provider call. A team at 100% receives a 402 with a reset date, not a silent fallback to a cheaper model. The fallback path is for provider failure, not for spending policy.

## Stack

FastAPI · LiteLLM · vLLM · OPA/Rego · Langfuse · OpenTelemetry · Postgres · ClickHouse · Kubernetes · Argo Rollouts.

The application-facing contract is a FastAPI adapter; LiteLLM is the routing layer behind it. The deep tier runs on self-hosted vLLM; the fast tier uses hosted providers.

## Key decisions

### 01 — One central gateway, not an SDK per application

**Decision.** All model traffic goes through one OpenAI-compatible endpoint. Applications keep their existing SDKs and point the base URL at the gateway.

**Why.** Policy, budgets, and audit are cross-cutting concerns. A gateway makes them enforceable once, and it gives six teams one place to change a provider without a release.

**Trade-off.** A single point of failure and 18 ms of added p95. Three replicas across zones, a 25 ms budget, and a degraded mode for provider outages address the first; the second is measured on every release.

**What would reopen it.** If a team needs a provider feature the gateway cannot express, the answer is a supported escape hatch with its own audit — not a second uncontrolled path. Revisit if the gateway becomes a feature bottleneck or its p95 overhead exceeds the budget for two consecutive releases.

### 02 — Policy in OPA/Rego, not Python

**Decision.** Policy lives in Rego, ships as a signed bundle, and is evaluated in a sidecar. Security owns the rules; the platform owns the runtime and the decision log.

**Why.** Policy changes on a different cadence from code. OPA gives a declarative language, unit-testable rules, bundle versioning, and decision logs that include the policy revision, the redacted input, and the redaction labels.

**Trade-off.** An extra network hop (5 ms p95) and a new language for the team. A policy outage must fail closed; the sidecar is replicated with the gateway and its bundle is cached locally.

**What would reopen it.** If policy shrinks to a small, stable set of checks owned by the same team, an in-process library may be simpler. That is a different governance model, not a performance tweak.

### 03 — Redact before policy, not after

**Decision.** Run deterministic redaction and label extraction before OPA. The policy receives the redacted prompt, its length, and the labels — never the raw clinical text.

**Why.** No raw PHI in logs was a hard requirement. Redacting first also means a provider can never receive a request that skipped the redactor, even if the policy path changes later. OPA's decision log contains the redacted prompt and labels, not raw clinical text.

**Trade-off.** A denied request still pays the redaction cost, and policy cannot inspect raw text for semantic cues. Semantic PHI detection remains a separate classifier with its own fail-closed path.

**What would reopen it.** If OPA can prove that decision logs never retain input, or a policy genuinely needs raw text to decide, revisit the order. Until then, privacy wins over a few milliseconds on denied traffic.

### 04 — Evals as a blocking gate, with a canary behind it

**Decision.** Every prompt or model change runs the 240-case clinical suite. No failures means the canary may start at 5%; Argo Rollouts moves to 25% and 100% only while quality, latency, and cost guardrails stay green.

**Why.** The suite catches known regressions before traffic; the canary catches the unknown. The gate is deterministic and auditable, and the rollout is reversible without a redeploy.

**Trade-off.** Releases take longer and the suite needs maintenance. The team reviews failures weekly, adds adversarial PHI cases every month, and keeps a reviewed exception path for hotfixes.

**What would reopen it.** If the suite's precision falls below the team's trust threshold, the gate becomes theatre. The answer is better cases and human review, not removing the gate.

## Measured outcome

| Measure | Result | Scope and limit |
|---|---|---|
| Cost per 1,000 requests | −47% | After task-class routing, caching for identical non-PHI requests, and eliminating retry storms across six applications |
| p95 gateway overhead | 18 ms | Including OPA and redaction; 25 ms budget; alerting fires at 22 ms |
| Policy audit coverage | 100% | Decision ID, trace ID, and policy version on every policy decision; redaction labels recorded alongside the request |
| Overhead split | 4 ms redaction, 5 ms OPA | The rest is routing and serialization |
| Scale | 9 models, ~40M requests/month | Six teams, three providers, 16 weeks |
| Audit retention | Seven years | ClickHouse holds the audit record and cost ledger |
| Release gate | 240-case clinical suite | Canary 5% → 25% → 100%; a failed guardrail rolls back |

The 47% is a cost result, but the audit result is the one that mattered. A security reviewer can answer "why was this allowed?" from a decision ID, a trace ID, and a policy version without reading a prompt. That is the difference between a gateway and a proxy.

## What would reopen the design

The gateway shape reopens if policy becomes a small, stable set of checks owned by one team (in-process may be simpler), if a provider offers in-region, no-retention inference at a comparable price (the self-hosted deep tier loses its main reason), or if the eval suite's precision falls below the team's trust threshold. Each is a governance change, not a latency tweak. The adapter contract is the escape hatch: the router can be replaced without changing six applications.

## Disclaimer

Anonymized prior work. Delivered in an earlier role, not as an Aldridge Systems engagement. Client identity, identifying details, exact dates, and commercially sensitive figures are withheld or rounded. The stack and policy model are described as built. Nothing here is a guarantee of future cost, latency, availability, or regulatory outcome.
