# Case 02 — The forecast was right. Monday disagreed.

**Demand forecasting · Logistics · 10 weeks · Lead MLOps engineer**

> **Anonymized prior work.** Delivered in an earlier role, not as an Aldridge Systems engagement. Client identity, identifying details, exact dates, and commercially sensitive figures are withheld or rounded. The stack and pipeline semantics are described as built; the figures are measured results from that engagement, not projections.

## Context

A multi-region replenishment operation: 1,800 SKUs across four regions, a daily batch, and a 15-minute source-to-publish freshness SLO. The gradient-boosted forecast beat the incumbent by 18% in backtests. In production, planners overrode 38% of recommendations and the Monday forecast was consistently low.

The model was fine. The features were not.

## The problem

Training joins used the time a row landed in the warehouse, not the time the event happened. The model learned from carrier delays that were not yet known at forecast time, promotions that landed after the nightly build, and a regional feed that had been stale for five weeks. The backtest was measuring a world that never existed.

Root causes, in order of impact:

1. Point-in-time leakage in the training joins: ingestion time used as event time.
2. Promotion calendar landed after the feature build; Monday missed weekend promotions.
3. Carrier-delay feed stale by 6–30 hours, with no freshness measurement.
4. No drift visibility: a regional carrier change shifted the feature distribution silently.
5. Retraining was a notebook run by one analyst, with no lineage, gate, or rollback.

The forecast did not need a smarter model. It needed a pipeline that could prove where every number came from — and stop the line when it could not.

## Constraints and non-goals

**Constraints**

- No new data platform: existing Postgres, Parquet on S3, and orchestration.
- Peak season in twelve weeks; planning cannot freeze for a migration.
- Planners need a reason for every changed number, not a global accuracy chart.
- Audit needs lineage from forecast to features to source partitions.
- Two data engineers, one planner; handover in the final two weeks.

**Non-goals**

- Replace the model. It was the one component that did not need replacing.
- Build a new warehouse or a second feature definition for batch scoring.
- Continuous retraining. A weekly release gate was the right cadence.
- A custom drift dashboard. Use the alerting the team already trusted.

## Shape

Five steps, one definition:

1. **Source / event time** — orders, carrier delays, promotions, and stockouts land as partitioned Parquet and Postgres tables. Every row carries the event timestamp, not just the ingestion timestamp.
2. **Offline / point-in-time** — Feast joins features as of each forecast timestamp. The training set is rebuilt from the same definitions that serve the daily batch, so leakage becomes a failing test rather than a silent bias.
3. **Assets / daily partition** — Dagster materialises `demand_features`, `forecast`, and `publish_forecast` for one date. Re-running a partition is idempotent; dependencies are explicit, not cron ordering.
4. **Checks / blocking** — freshness, coverage, non-negative demand, and a forecast-band check run before publish. A failed blocking check leaves planners on the last approved partition.
5. **Drift / paper trail** — Evidently compares the current feature window with the reference; Prometheus stores the drift share by region; Grafana pages the planner on-call with the region and the source feed.

The subtle part is that the offline and online paths share one definition. Feast's point-in-time join scans backward from each entity timestamp for at most the feature view's TTL, so the training row sees exactly what the model would have seen at that moment — not what the warehouse knew later.

Publishing is a separate asset for a reason. A forecast can be computed and still be wrong to ship. The publish step is the only thing planners consume, and it only runs when the blocking checks pass.

## Stack

Python · Dagster · Feast · Evidently · Prometheus · Grafana · Postgres · Parquet/S3.

One feature contract, `replenishment_v3`, with a 36-hour TTL. Daily partitioned assets; weekly retraining with a shadow week.

## Key decisions

### 01 — Feast feature views, not a hand-rolled join

**Decision.** Define each feature once in Feast, with event timestamps, entities, TTL, ownership, and a freshness SLO. Use the same definition for historical retrieval and online reads.

**Why.** Point-in-time correctness was the missing property. A feature store makes it the default rather than a code-review checklist, and the TTL forces the team to state how far back a feature is allowed to look.

**Trade-off.** A new component to operate, and schema discipline on every change. The team already had Postgres and S3; Feast added a registry and a serving path, not a new warehouse.

**What would reopen it.** If the pipeline became purely batch with one consumer and no online reads, a tested SQL join with the same contract could be enough. The value was parity between training and serving, not the feature store logo.

### 02 — Dagster assets with blocking publish checks

**Decision.** Model the pipeline as daily partitioned assets with explicit dependencies. Freshness, coverage, non-negative demand, and a 25% band check block `publish_forecast`. A failure leaves the last approved partition in place.

**Why.** Assets describe what exists, not just what runs. A failed check has a downstream consequence, a partition can be replayed, and lineage is recorded as metadata instead of reconstructed from filenames. Blocking is the point: a freshness check that only colours a dashboard is advisory.

**Trade-off.** A new scheduler and UI for the team to learn, and false positives can stall a publish. The team ran the checks in shadow mode for two weeks, tuned the band, and added an override that records a reason and an approver.

**What would reopen it.** If the organisation already runs a mature orchestrator with asset checks, adding a second one is the wrong move. If the business needs a forecast even when a source is late, the answer is a degraded forecast with an explicit confidence flag — not a disabled check.

### 03 — Evidently and Prometheus, one labelled series per region

**Decision.** Use Evidently's data-drift preset with an explicit PSI method and a 30% dataset threshold. Export one labelled gauge per region; alert and dashboard read the same series.

**Why.** Evidently covers numerical and categorical drift with tested statistical methods; Prometheus already had the alerting and retention the team trusted. The integration is a gauge, not a new observability stack.

**Trade-off.** Statistical drift is not business impact. A 0.31 share can be noise in a low-volume region; a 0.12 shift can be a real carrier change. The alert is a prompt to investigate, not a verdict.

**What would reopen it.** If drift alerts repeatedly fail to predict planner overrides, the threshold should be learned from override reasons rather than tuned by hand. That is a labelling project, not a dashboard project.

### 04 — Weekly retraining with a seven-day shadow

**Decision.** Retrain weekly. Backtest on the last eight weeks, run the candidate in shadow for seven days, and promote only if WAPE improves by at least 2% and bias stays within ±3%.

**Why.** Retraining is a release, not a cron job. The shadow week catches distribution surprises that a backtest cannot, and the gate makes the promotion decision explicit and reversible.

**Trade-off.** A week of lag during a sudden shift. The drift alert and the planner override channel provide the early warning; a promotion can be pulled forward with an incident review.

**What would reopen it.** A promotion-heavy business with event-driven demand may need a separate event model, not a faster global retrain. The weekly gate would remain for the baseline model.

## Measured outcome

| Measure | Result | Scope and limit |
|---|---|---|
| WAPE | 24.6% → 17.0% | 31% relative reduction over the first eight weeks; the model changed once, the feature path changed completely |
| Excess inventory | −22% | Across four regions; stockouts stayed flat; the gain came from fewer panic overrides, not a tighter service level |
| Feature freshness | 12 minutes p95 | Against a 15-minute SLO |
| Planner overrides | 38% → 22% | Every override now carries a reason code that feeds the next feature review |
| Detection | 36 hours | A regional carrier change was caught by the drift alert, not five weeks later by a spreadsheet |
| Delivery | 10 weeks | Four regions, before peak season |

The forecast did not get smarter. It stopped seeing the future. The model changed once; the feature path changed completely. That is the uncomfortable lesson: the backtest was not wrong about the model; it was wrong about the world.

## What would reopen the design

The design reopens if the pipeline becomes purely batch with one consumer (Feast may be unnecessary), if the platform standardises on another orchestrator with blocking data-quality checks, or if promotions become a material share of demand and a weekly cadence cannot react. The common thread is the contract, not the tool: the thing worth keeping is the agreement about what a feature means at forecast time.

## Disclaimer

Anonymized prior work. Delivered in an earlier role, not as an Aldridge Systems engagement. Client identity, identifying details, exact dates, and commercially sensitive figures are withheld or rounded. The stack and pipeline semantics are described as built. Nothing here is a guarantee of future forecast accuracy, inventory performance, or freshness.
