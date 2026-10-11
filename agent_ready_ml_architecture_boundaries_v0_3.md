# Architecture boundaries — review draft v0.3

**Companion to MVP spec v0.6. No implementation or scaffolding authorized.**

## Rule
One conceptual owner per invariant. Start with one Python distribution. Boundaries do not mandate folders, classes, or generic interfaces.

| Boundary | Owns | Must not own |
|---|---|---|
| Contracts | Feature, prediction, metric, artifact semantics and compatibility | Source access or reports |
| Data/features | Discovery, bounded profiling, ingestion, SQL transforms, snapshots | Model qualification |
| Models | Training, import, artifacts, inference compatibility, activation | Source-specific connectors |
| Evaluation | Metric calculations, baselines, backtests, experiments | Marimo visualization or deployment approval |
| Policies | Versioned policy resolution, gates, approval records | Agent interpretation or generic rules engine |
| Execution/metadata | SQLite run and active state, manifests, finalized artifact compatibility, minimal direct operations | Arbitrary DAGs, scheduler, distributed workers, competing Dagster dependency logic |
| Predictions | Canonical records, query and provenance | Destination-specific output schema |
| Optional integrations | Marimo, Dagster, PostgreSQL publishing | Core metric calculations or model state |
| Agent guidance | Discovery, questions, interpretation, reports | Runtime enforcement |

## Dependency rules
Core contracts are shared; domain components use their published contracts. Execution coordinates public domain operations and cannot replace them. Optional integrations call core operations; core cannot depend on Marimo or Dagster. Agent skills are not the source of policy truth. Avoid circular dependencies and duplicate paths for training, evaluation, or activation.

## Storage boundaries
DuckDB handles analytical queries. Parquet holds persisted feature datasets, immutable training snapshots, and default prediction data. SQLite handles transactional operational state, active versions, experiments, policy references, and run records. Model binaries live in artifact storage. Do not build an abstract backend marketplace without a second demonstrated backend.

## Invariants and change control
Semantic feature changes require compatibility review. Activation must be atomic and authorized. Canonical predictions precede optional external publication. Evaluation results, not notebooks, are authoritative. Only verified completed artifacts are reusable. Every proposed new module needs a user scenario, one owner, an input/output contract, permitted dependencies, and a verifiable failure case.

## Anti-slop review
Reject speculative adapters, general-purpose workflow DSLs, duplicate registries, hidden Python feature paths, all-purpose `utils`, notebook-owned metrics, premature microservices, and parallel integration-specific implementations. Do not optimize for arbitrary file count; optimize for few comprehensible concepts.

## Resolved architectural directions
Immutable resolved manifests and verified artifact finalization; semantic feature revisions with pinned model references; qualification separate from activation authority; PostgreSQL publishing; toolkit-owned artifact compatibility and Dagster-owned orchestration. Exact implementation-level acceptance details remain in the freeze-review flags.

## Consolidated authority and delivery contracts
Definitions in project files are authoritative for intent. Immutable resolved execution manifests pin the actual definitions, inputs and relevant policy revisions used by a run, including uncommitted edits. Finalized artifacts hold evidence; SQLite owns operational state. Reports are derived views.

Prediction generation finalizes a canonical batch before optional external publication. Each destination has its own delivery attempts and status (including partial and unknown). Retry must reuse prediction identity and be automatic only when idempotency or transactional safety is established.

Historical replay owns prediction/outcome alignment and declared temporal validation, not domain-specific settlement or counterfactual truth. Agents and optional adapters may not implement alternative metric, policy or lifecycle logic.

Agent instructions are package-owned and version-matched, with minimal reversible registration; no overwrite of existing project guidance.

## Final contract gates
All eight product-level gates are resolved in MVP v0.6. Only the explicitly identified bounded acceptance details remain before implementation authorization.

## Revised Dagster boundary (supersedes native-first language above)
Dagster is the **preferred** coordinator for multi-stage/recurring workflows and owns asset dependencies, scheduling, retries and rematerialization when enabled. It is optional. Direct execution invokes only individual toolkit operations and the fixed golden path. The toolkit owns artifact finalization, compatibility decisions, ML validity, authorization and canonical prediction status. A successful Dagster materialization is not proof of toolkit artifact validity. No second native DAG engine or retry scheduler.

## Finalized gate constraints
First-party data: CSV, Parquet, DuckDB and read-only PostgreSQL. Model family: XGBoost only. PostgreSQL is the only first-party external publisher. History append-only by default with semantic current-selection and retention controls. Temporal contracts require event, prediction and availability semantics. Backtests support chronological holdout and expanding walk-forward, with fold evidence and explicit decision assumptions. Claude Code and Codex are reference agents; Marimo notebooks must pin immutable evidence.

## Freeze-review flags
Specify exact manifest/fingerprint fields, crash reconciliation across files and SQLite, supported safe XGBoost import formats, PostgreSQL upsert preconditions, and the minimal Dagster acceptance surface. These are specification details, not new architecture.