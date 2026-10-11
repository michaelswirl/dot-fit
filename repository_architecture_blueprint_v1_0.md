# Repository Architecture Blueprint v1.0
**Status:** Proposed for approval; no implementation authorization  
**Companions:** MVP Specification v0.6; Architecture Boundaries v0.3; Agent Operating Contract v0.3  
**Scope:** Repository design, consuming-project conventions, ownership, dependency governance, and acceptance checks. No source code, tickets, or scaffolding.

## 1. Governing decisions

- **R1** Standard `ml/` workspace within an existing application; do not impose an application-wide structure.
- **R2** User-owned, local-first artifact storage, with S3-compatible object storage and verified migration; operational metadata is separate.
- **R3** One shared artifact identity, finalization, integrity, and location contract; domain components validate content.
- **R4** Internal domain ownership; a small workflow-oriented public interface.
- **R5** Task-oriented, package-owned skills with shared authoritative references.
- **R6** Conventions and one small project manifest; secrets external.
- **R7** Automated dependency/contract checks plus review of conceptual duplication.
- **Decision 6 (revised)** Dagster preferred for orchestrated workflows; direct execution minimal and Dagster optional.

## 2. Two distinct repository surfaces

**Toolkit source repository**: where contributors maintain contracts, runtime behavior, adapters, documentation, and skills.  
**Consuming application workspace**: where users declare their own ML sources, shared features, models, metric definitions, and policies. These are *not* interchangeable.

### 2.1 Proposed toolkit repository layout (ownership map, not scaffolding)

```text
toolkit-repo/
  README.md
  LICENSE
  pyproject.toml
  AGENTS.md                         # contributor-only guidance
  docs/
    product/                       # authoritative product scope
    architecture/                  # boundaries, diagrams, review policy
    contracts/                     # public behavioral contracts
    decisions/                     # ADRs, including superseded decisions
  skills/
    discover-data/
    build-model/
    evaluate-and-backtest/
    integrate-predictions/
    diagnose-ml-workflow/
    references/                    # canonical cross-skill guidance
  src/<package>/
    contracts/                     # identities, compatibility semantics
    data/                          # discovery, source reads, bounded profiling
    features/                      # SQL definitions, snapshots, temporal checks
    models/                        # XGBoost training/import, inference, activation
    evaluation/                    # metrics, folds, backtests, experiments
    predictions/                   # canonical history and current selection
    policies/                      # definitions, resolution, gates, approvals
    artifacts/                     # storage identities, integrity, migration
    operations/                    # public workflow-facing entry points
    state/                         # SQLite operational records, manifests
    integrations/
      dagster/                     # asset orchestration; no ML rules
      marimo/                      # evidence consumption/report helpers
      postgres/                    # authorized publisher
  tests/
    contracts/
    boundaries/
    scenarios/
  examples/
    tabular-classification/
    time-dependent-forecast/
    import-xgboost/
```

**Important:** Each directory is provisional. Create a module only after its first cohesive responsibility is demonstrated. No mandatory `utils/`, `base/`, `adapters/` hierarchy or one class per noun. The `<package>` name remains undecided. The list of skills is a candidate minimal set, not a requirement to create five files before their workflows are specified.

### 2.2 Proposed consuming application layout

```text
existing-app/
  ...existing application files unchanged...
  ml/
    project.<declarative-format>      # one small project manifest
    features/                         # reusable SQL + semantic contracts
    models/                           # model definitions and feature references
    metrics/                          # reusable metric definitions
    policies/                         # versioned qualification / publishing rules
    reports/                          # intentionally retained Marimo sources
  ...minimal agent discovery registration...
```

The workspace holds *intent*. Generated snapshots, model binaries, prediction history, run manifests, and metric evidence are outside Git-tracked definitions by default. Local managed storage may physically be a project-associated ignored directory; the path is a configuration detail, not part of logical artifact identity.

## 3. Ownership map

| Owner | Authoritative responsibility | Explicit exclusion |
|---|---|---|
| Contracts | IDs, schemas, semantic revision rules, compatibility | Running SQL, calculating metrics |
| Data/features | Read-only discovery, bounded profiling, SQL, feature snapshots | Training, qualification |
| Models | XGBoost train/import, artifact-specific validation, inference, activation | Source connectors, business metrics |
| Evaluation | Metric definitions/calculations, temporal folds, replay, experiment evidence | Marimo, production authorization |
| Predictions | Canonical append-only records, semantic current-selection | PostgreSQL destination schema |
| Policies | Policy IDs/revisions, resolution, deterministic gates, approval evidence | Agent explanations, generic policy DSL |
| Artifacts | Location, stable identity, verified finalize, read/verify/migrate | Semantic feature or model validation |
| State | SQLite mutable run state, active pointers, delivery attempts, manifest indexes | Source definitions |
| Operations | Thin public coordination over owners | Duplicate training/evaluation logic |
| Integrations | Dagster scheduling/graph, Marimo display, PostgreSQL delivery | Alternative ML contract implementation |
| Skills | Agent workflow, clarifying questions, interpretation | Enforcing correctness |

**No second owner:** A rule that cannot be assigned to exactly one row is a review finding, not an invitation to add a shared helper.

## 4. Dependency direction and enforcement

- `contracts` is foundational and must not import domain implementations or integrations.
- Domain modules depend on published contracts, not on each other's private implementation details.
- `operations` coordinates public domain functions; it is not a new implementation of their logic.
- `integrations` may call operations and public contracts. Core domains must not import Dagster or Marimo.
- `skills` are distributed alongside package versions but do not execute as an enforcement engine.
- `state` records mutable status and references; it cannot replace authoritative version-controlled definitions.
- No import cycles; no direct feature-to-Dagster or metric-to-Marimo dependency.
- PostgreSQL source read access and PostgreSQL destination write access are separate authorization surfaces.

**Enforcement plan:** fast import-boundary/cycle checks, optional-dependency isolation checks, public contract tests, and scenario-level integration tests. Review separately for duplicate concepts and unnecessary abstractions. Changing enforcement rules requires deliberate review.

## 5. Public operation surface

Proposed user-intent operations: **discover**, **prepare features**, **train/import**, **evaluate**, **backtest**, **predict**, **qualify/activate**, **publish**, and **inspect/recover**. These are conceptual operations, not committed CLI commands or method names.

Each operation must have a documented input contract, output/evidence contract, failure statuses, authority requirements, and artifact references. Dagster and direct execution invoke the same operations.

**Direct path:** individual operations plus a fixed golden-path sequence; no arbitrary DAG, cron, generalized retry queue, or separate scheduler.  
**Dagster path:** asset dependencies, materialization coordination, scheduling, retries, sensors, and workflow-level recovery. A Dagster success event cannot override failed toolkit validation.

## 6. Project manifest boundary

The single manifest may identify workspace paths, named source profiles, artifact storage selection, operational state location, default execution preference, and references to model/policy definitions. It must **not** duplicate feature schemas, metric formulas, policy thresholds, model training configuration, or secrets.

Resolution order is a freeze-review detail. The required property is a deterministic, inspectable *effective configuration* that a run manifest can pin without capturing credentials.

## 7. Artifact storage and state

**Default:** user-owned local managed artifacts and local SQLite operational metadata.  
**Portable artifact backend:** conservative S3-compatible object operations; user-owned buckets.  
**Not implied:** multi-host shared SQLite, distributed locking, concurrent workers with one local state file, or object-store transactions.

Artifact lifecycle: stage -> domain-validate -> integrity-verify -> finalize -> reference. Never expose partial artifacts as finalized. Finalized artifacts have stable logical identity and fingerprints independent of local/S3 URI.

Migration: enumerate finalized referenced artifacts; copy to user-owned destination; verify fingerprints; update location mapping safely; preserve source until migration is confirmed; recover cleanly from partial migration. **Do not claim local-to-S3 migration automatically moves SQLite state or running jobs.**

**Authority:** project files define intent; immutable resolved manifests identify executed inputs/revisions; finalized artifacts preserve evidence; SQLite records mutable status, active version, and delivery attempts.

## 8. ML workspace and agent behavior

Canonical task skills are package-owned and version-matched. Claude Code and Codex get minimal reversible discovery registration; neither receives an independently maintained copy of the behavioral rules.

Agents inspect existing application structure before adding `ml/`, never overwrite existing agent guidance, and ask about consequential source/target/time ambiguity. They recommend SQL, use the toolkit's public operations, and produce Marimo reports pinned to immutable evidence. Reports cannot become the source of metric truth.

A standard workspace is recommended, not a requirement to relocate existing application code or rewrite existing database tables.

## 9. Cross-document contract checks

| Scenario | Required invariant |
|---|---|
| SQL feature semantics change | New immutable revision; dependent models remain pinned |
| Training data changes | Manifest changes; incompatible checkpoints refused |
| Dagster retries failed evaluation | Finalized compatible training artifact may be reused |
| Inference succeeds, PostgreSQL export fails | Canonical predictions remain valid; delivery state separate |
| Agent requests activation | Qualification and scoped approval independently verified |
| Marimo report reopened | Reads pinned evidence; no silent live-data recalculation |
| Local artifacts moved to S3 | Logical IDs and fingerprints unchanged |
| Second agent adopts project | Same canonical guidance and public contracts |

## 10. Architectural anti-slop checklist

A proposed module or abstraction must answer: Which user scenario requires it? Who owns it? What published contract does it expose? What is its failure case? Why can't an existing owner handle it? Why is a new dependency justified?

Reject speculative adapters, redundant registries, alternate evaluation engines, general-purpose workflow DSLs, hidden Python-only feature paths, notebook-calculated authoritative metrics, per-model duplicated shared features, and premature distributed services.

No arbitrary file-size quotas or forced class hierarchies. Favor fewer cohesive concepts over a larger number of tiny wrappers.

## 11. Independent cross-document review — findings

**Aligned:**
1. Dagster is preferred but optional; no competing native orchestrator.
2. S3 portability applies to immutable artifacts, not live SQLite transactional state.
3. Agent skills are canonical/versioned, while runtime enforces safety.
4. Predictions are canonical before any external publishing.
5. Marimo consumes evidence rather than calculating authoritative evaluation results.
6. Model-family support is XGBoost only in the initial release; data support is CSV, Parquet, DuckDB, and read-only PostgreSQL.

**Remaining freeze details, not permission to improvise:**
- Actual package/workspace manifest naming and configuration precedence.
- Exact artifact fingerprint/manifests and crash reconciliation semantics.
- Local/S3 supported operations and verified migration failure tests.
- PostgreSQL publisher upsert preconditions and unknown delivery handling.
- Claude Code/Codex registration behavior in pre-existing repositories.
- Minimum Marimo evidence schema and temporal uncertainty estimator applicability.
- Explicit deployment note: S3 artifact storage does not make SQLite multi-writer/distributed.

**Review verdict:** Architecture direction is internally consistent and ready for *blueprint approval*. It is **not** implementation authorization until the bounded freeze details are made testable and accepted.

## 12. Change control

Approve this blueprint as the ownership reference, then resolve the listed freeze details in the relevant contract documents. Do not start coding or ticketing solely because the directory tree exists here. New architectural concepts require a documented user journey and an ADR explaining why an existing owner cannot satisfy it.