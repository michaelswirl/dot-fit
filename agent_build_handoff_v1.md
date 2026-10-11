# Agent Build Handoff — First Vertical Slice

**Status:** Authorized scope proposal; user may launch agent after review. This does not authorize full MVP implementation.
**References (read before any edits):** `agent_ready_tabular_ml_mvp_spec_v0_6.md`, `agent_ready_ml_architecture_boundaries_v0_3.md`, `agent_ready_ml_agent_operating_contract_v0_3.md`, `repository_architecture_blueprint_v1_0.md`.

## Mission
Build the smallest end-to-end, verifiable SQL-first tabular ML slice on synthetic churn data. Preserve the approved architecture without creating empty placeholder subsystems. Start with an architecture/acceptance plan and then implement only this slice. Do not treat every section of v0.6 as first-task scope.

## First-slice user story
A developer with synthetic customer, subscription, and transaction data can ask an agent to inspect the dataset, identify ambiguities, define SQL features, create an immutable training snapshot, compare a simple baseline against XGBoost, generate an evaluation evidence artifact, and write canonical batch predictions to local Parquet. A different application can query those predictions using DuckDB. No external writes or credentials.

## Fixture design (synthetic only)
Three tables: `customers(customer_id, joined_at, segment)`, `transactions(transaction_id, customer_id, event_at, available_at, amount)`, `subscriptions(customer_id, status_at, available_at, status)`; plus a separately available labeled outcome table with `customer_id`, `prediction_at`, `label_available_at`, and `churned_in_next_30d`. Include 1,000–5,000 customers and multiple monthly cutoffs, enough positive labels to evaluate. Seeded generation must be deterministic. All times UTC.

The fixture must deliberately contain: duplicated transaction IDs, a late-arriving transaction, missing amount values, one status update after a prediction cutoff, a customer with no transactions, and one deliberately leaky feature definition used only in a negative test. Define whether the churn label represents an event during the next 30 days and ensure label availability is later than the outcome window.

## Feature contract
One row per `(customer_id, prediction_at)`; SQL-first features: account age at cutoff, 30-day transaction count, 30-day spend, latest known subscription status. Only records with `available_at <= prediction_at` are eligible. Document null handling and feature meaning. Train and infer from the same pinned feature revision. For a training fold, exclude labels not available by its training cutoff.

## Build boundary
**In scope:** bounded local discovery/profile summary, synthetic fixture, one feature definition + semantic contract, immutable local Parquet snapshot with verified finalization, baseline, XGBoost training, chronological holdout evaluation, canonical prediction dataset, minimal run manifest, automated positive and negative tests. Simple user-facing Markdown evaluation summary is enough for this first slice.

**Explicitly out of scope:** S3 backend, storage migration, Dagster integration, PostgreSQL publishing, production activation, Marimo notebook generator, walk-forward backtesting, external model import, HTTP API, hosted service, generic connector/plugin systems, arbitrary DAGs, full policy DSL, multi-agent acceptance. These remain in the approved MVP roadmap; do not create placeholders for them.

## Acceptance scenarios
1. **Clean path:** seeded fixture → profile → SQL feature snapshot → baseline/XGBoost evaluation → canonical Parquet predictions, with a stable run manifest and all required checks passing.
2. **Temporal leakage:** deliberately leaky SQL/late-arriving record is rejected or explicitly flagged as unverified before a passing historical evaluation; no false claim of leakage-free proof.
3. **Schema drift:** rename or change a required feature type; inference must fail with a clear contract mismatch.
4. **Reproducibility:** same fixture seed and pinned definitions reproduce input identities and compatible results (numeric tolerance allowed where justified).
5. **Interrupted artifact:** simulate incomplete snapshot/model write; incomplete output must not be reusable or appear finalized.
6. **Boundary discipline:** no Dagster/Marimo imports in core; no second metric implementation; no optional integration dependency required for the first slice.
7. **Consumer independence:** a non-Python consumer can read the canonical predictions through a documented DuckDB SQL query or Parquet interface without importing Python into its web runtime.

## Quality gates
Run deterministic tests and report commands, results, and unverified assumptions. Ensure feature SQL never includes future availability. Use task-appropriate metrics (log loss and Brier for probabilistic binary classification, plus baseline comparison and sample counts). Do not claim improvement without evidence. No network access or production data required. Avoid unsafe model deserialization.

## Stop conditions
Stop and ask for approval if a requirement implies a new package, general storage abstraction beyond local artifacts, external credentials, application schema modifications, distributed execution, a new policy language, or changes to approved architecture boundaries. Stop if the fixture cannot support meaningful temporal evaluation; fix the fixture design first.

## Agent execution protocol
1. Read the four references; summarize the slice, explicit exclusions, and planned module ownership.
2. Propose the smallest concrete file tree and acceptance tests; note deviations from blueprint.
3. Implement only the approved slice in small, verifiable steps.
4. Run tests, architecture checks, and one end-to-end example.
5. Provide a concise handoff: files changed, commands, test outcomes, artifacts produced, known limitations, and next proposed slice. Never declare full MVP complete.

## Launch prompt (copy into coding agent)
> Read `agent_ready_tabular_ml_mvp_spec_v0_6.md`, `agent_ready_ml_architecture_boundaries_v0_3.md`, `agent_ready_ml_agent_operating_contract_v0_3.md`, `repository_architecture_blueprint_v1_0.md`, and `agent_build_handoff_v1.md` in that order. The repository architecture blueprint is approved. Implement ONLY the first vertical slice defined in the handoff using synthetic data. Before edits, present the smallest file layout, ownership map, and tests. Follow the stop conditions and do not invent abstractions for deferred integrations. After implementation, execute the tests and end-to-end scenario, and report exact outcomes. Do not implement the entire MVP, create speculative scaffolding, or claim production readiness.