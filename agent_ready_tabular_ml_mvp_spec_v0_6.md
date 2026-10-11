# Agent-ready tabular ML toolkit

**MVP specification v0.6 — final consolidation candidate**  
**Status:** Not approved for implementation  
**Project:** Open-source first; no cloud dependency

## 1. Purpose

Help coding agents build and maintain tabular predictive features in existing applications without inventing an MLOps stack. The toolkit supplies data contracts, standard ML operations, validation, reproducible evaluation, and access to predictions. Agents investigate data, propose changes, interpret evidence, and generate Marimo reports. Deterministic software enforces the contracts and production permissions.

The toolkit is independent of Claude Code, Codex, Pi, and any hosted agent provider. Agent-specific discovery mechanisms may differ, but they must use the same authoritative contracts.

## 2. Who it is for

The primary audience is developers using coding agents to build tabular predictive applications, including independent quantitative researchers and application builders. A second audience is developers integrating existing trained models. The core must remain domain-independent: application-specific business rules belong in user-defined policies and extensions.

The first complete user journey is **raw data → evaluated model → predictions**. The user can access standardized predictions locally or arrange authorized publication into an existing database or API. The Next.js application does not need a Python runtime.

## 3. Confirmed discovery decisions (22)

1. The first complete journey covers data discovery, feature engineering, training, evaluation, and predictions.
2. Feature engineering is SQL-first. Python transformations are allowed but must meet the same contracts.
3. Training uses immutable, identifiable feature snapshots. Inference may read directly or use materialized datasets.
4. Evaluation includes standard metrics, baseline comparisons, declared qualification gates, and agent-guided interpretation.
5. Agents may conduct approved research autonomously. Production activation and external publication require authorization or a previously approved policy. Broader policy-governed autonomy is a future option.
6. Dagster is the preferred orchestrator for multi-stage and recurring workflows, but optional. Minimal direct execution runs the fixed golden path and individual operations. Both invoke identical toolkit operations; the toolkit owns correctness and compatibility.
7. DuckDB-native sources are preferred. Other sources enter through a standard ingestion boundary.
8. Evaluation produces structured evidence. Agents generate Marimo notebooks as the default human-facing report, without recalculating authoritative metrics.
9. Agent guidance is portable. The runtime and contracts do not depend on an agent provider.
10. Time-dependent models require declared prediction-time and feature-availability semantics. No automatic check can prove that all leakage is absent.
11. Feature contracts cover structure and meaning; operational quality and qualification policies are separate.
12. Structural safety and a declared qualification policy govern activation. Predictive and real-world metrics are both eligible as policy gates.
13. Predictions have a canonical representation and standardized access. External database/API publication is optional and cannot redefine the internal prediction contract.
14. Existing models use guided import with progressive qualification. Missing provenance is disclosed, never invented.
15. Experiments are lightweight records. Historical backtesting is first-class and recommended for time-dependent models, not universally mandatory. Decision-policy backtests and business-value metrics are encouraged. Marimo is the default exploration interface.
16. Operational metadata uses a lightweight transactional store, provisionally SQLite. DuckDB and Parquet handle analytical datasets.
17. Policies are declarative, project-owned, versioned and reusable; SQLite records applied revisions. Distinct policy categories share resolution and audit conventions without a generic rules engine.
18. Safe checkpointed recovery reuses completed artifacts only when verified input, contract and dependency identities remain compatible.

## 3A. Decisions 19–22 and enforceable consequences

19. **Immutable resolved execution manifests:** Project files define intent, but each run pins the actual definitions, inputs, parameters, environment identifiers and applicable policy revisions used, including uncommitted changes. SQLite indexes run state; immutable artifacts retain evidence. Manifests must not contain secrets.
20. **Independent publishing with safe retries:** A canonical inference batch remains valid if external publishing fails. Publishing attempts have separate states, including partial and unknown delivery. Retrying uses the same prediction identities; automatic retries require verified idempotency or transaction support. Exactly-once API delivery is not promised.
21. **Historical replay and decision policies:** The framework aligns historical predictions and outcomes, enforces declared temporal constraints and provides a domain-neutral decision-policy boundary. Applications define actions, abstention, costs, and outcome assumptions. Recorded historical predictions must be distinguishable from retrospectively reconstructed predictions.
22. **Canonical package guidance with minimal registration:** The installed toolkit owns version-matched instructions and skills. Integration with existing repositories is non-destructive and reversible; Claude Code and Codex must each pass the same reference acceptance scenario. Agent instructions never replace runtime enforcement.

**Authority rule:** Version-controlled definitions own intent; resolved manifests record what actually ran; finalized artifacts own immutable evidence; SQLite owns mutable operational status and active-version pointers; Marimo is a derived view.

## 4. The canonical data path

A source may be CSV, Parquet, DuckDB, or read-only PostgreSQL. Unsupported APIs may feed the standardized ingestion boundary but have no built-in connector. The toolkit inspects metadata and bounded samples first. The agent reports observed facts, interpretations, and questions. It must obtain clarification when entity identity, target meaning, or data availability would materially change the model.

Shared feature definitions exist independently of models. DuckDB SQL is the default for joins, aggregates, time windows, and other tabular transformations. Python is an explicit extension for transformations that are clearer or impractical in SQL; Polars is optional, not a second required feature engine.

A feature contract declares the entity key, row grain, required fields, types, nullability, feature meaning, source provenance, and temporal meaning when applicable. A semantic change cannot silently masquerade as a compatible schema change. Validation can verify structure and declared rules; business meaning may require human confirmation.

Training and inference select compatible features from these definitions. `train.sql` and `infer.sql` are proposed naming conventions, not yet approved public interfaces. Training uses an immutable Parquet snapshot and records its provenance. Inference can query supported sources directly or use materialized features. The framework must reject incompatible model inputs in either path.

## 5. Discovery and source safety

Discovery begins with metadata, bounded samples, and targeted quality profiling. Reports distinguish observations, inferences, and user-confirmed semantics. Checks may cover nulls, duplicate keys, time coverage, schema changes, freshness, and unusual distributions.

Read-only source permissions are the default. The framework enforces supported read-only modes and least-privilege credentials; agent instructions alone do not constitute enforcement. Query budgets, timeouts, and privacy-conscious summaries prevent unbounded scans and unnecessary exposure of sensitive rows. Unsupported APIs may use a documented ingestion operation that produces a validated dataset.

**Ratified source scope:** The first relational source is read-only PostgreSQL. CSV, Parquet, and DuckDB are also supported. No built-in external API connector is promised.

## 6. Modeling and model import

The default training workflow establishes a task-appropriate simple baseline and trains an XGBoost candidate. The agent may recommend a different estimator with an explicit reason, but non-XGBoost candidates are outside guaranteed MVP support. There is no general AutoML search in the MVP. The system supports conventional tabular classification and regression; time-dependent tabular prediction must honor temporal contracts.

Existing model artifacts may be registered with incomplete training history. The agent identifies expected inputs and missing evidence. Production qualification still applies. Untrusted serialized artifacts must not be automatically executed during discovery. MVP import support is limited to supported, explicitly trusted XGBoost artifacts; arbitrary pickle or executable serialized files must not be loaded automatically.

Each model has at most one active version. Activation is atomic, preserves previous versions for rollback, and cannot expose incomplete artifacts. Concurrent activation requests cannot create two active versions.

## 7. Metrics, evaluation, and qualification

A metric definition states its calculation, required inputs, direction, and applicable context. An evaluation result records its observed value, dataset, model version, sample size, time period, and method. A qualification policy decides which metrics are gates, warnings, or monitoring signals.

Predictive metrics and real-world outcome metrics are equally eligible for qualification policies. Examples include log loss, Brier score, calibration, MAE, hit rate, ROI, and cost savings. Business metrics are encouraged, not required. Standard evaluation and an appropriate baseline are required before a newly trained candidate qualifies. There are no universal numerical performance thresholds.

The agent helps the user choose metrics and policies, explains limitations, and recommends experiments. It cannot override deterministic blocking checks or fabricate missing evidence. Monitoring can reuse metric definitions, but continuous monitoring and alert infrastructure are outside the MVP. Outcome-dependent metrics may only be computed after labels or realized outcomes become available.

## 8. Backtesting and experiments

Backtesting is a first-class historical evaluation operation. It supports timestamped predictions, realized outcomes, chronological holdout and expanding walk-forward evaluation, and optional decision policies. It records evaluation periods, sample sizes, costs and assumptions when supplied, and uncertainty or instability where estimable.

For time-dependent models, the agent recommends backtesting and explains the consequence of skipping it. A valid time-aware out-of-sample evaluation is still required for model qualification; a full historical decision-policy backtest is not. A model-specific qualification policy may make a backtest metric mandatory.

Decision policies are distinct from predictive models. The same predictions can be assessed under different policies without retraining. Historical replay must not be presented as proof of causal business impact or future profitability.

Recorded-at-the-time and retrospectively reconstructed predictions are labeled distinctly. Decision policies can abstain and may use domain-supplied cost and outcome calculations. A historical simulation is not automatically a causal estimate.

An experiment groups a hypothesis, comparable runs, baseline, datasets, evaluation rules, findings, and limitations. The MVP does not include a sweep engine, experiment server, or optimization dashboard. The framework must distinguish exploratory comparisons from final holdout evidence.

## 9. Predictions and application integration

Every inference run produces predictions satisfying one canonical contract. It records prediction identity, entity and target, model/version, prediction context and timestamp, value, and run provenance. The contract must support multiple legitimate predictions for the same entity at different times.

Predictions are queryable through a standard local interface; Parquet is the preferred durable format. An application may read them directly or authorize an export to PostgreSQL; HTTP publishing is deferred. Destination-specific field mappings and delivery rules sit outside the canonical prediction contract. External publication requires explicit authority or an approved policy.

One acceptance example is asynchronous batch predictions consumed by a Next.js application through its existing database. This example does not define a framework-specific dependency. The first publishing adapter is controlled PostgreSQL publishing. Canonical prediction history is append-only by default, with explicit retention rules and a current-prediction query respecting target, horizon, and validity semantics. Publishing attempts are recorded independently of inference. Failed or partially confirmed delivery does not invalidate canonical predictions; unknown delivery is not treated as success. Automatic retry requires safe idempotent or transactional semantics. Canonical predictions remain queryable regardless of whether external publishing is configured. Do not promise universal exactly-once delivery.

## 10. Declarative policies and operational authority

Policy definitions are human-readable, project-owned, and version-controlled. Each has a stable identity and immutable revision; models explicitly reference applicable policies, which may be shared. SQLite records exact policy revisions used by evaluation, qualification, activation, and publishing decisions. Missing mandatory policies block production operations. Agents may propose changes but may not silently weaken production safeguards. Policy categories include data quality, qualification, publishing, and monitoring; they share identity and audit conventions but not a generic rules language. Metric definitions are separate from policies: either predictive or real-world metrics may be gates, diagnostics, or monitoring signals. Monitoring checks may run when observations become available without requiring a background service.

## 11. Execution and operational state

Dagster is the preferred orchestrator for multi-stage and recurring workflows and owns asset dependencies, scheduling, sensors, retries, and rematerialization coordination when enabled. Minimal direct execution invokes individual toolkit operations and a fixed golden-path sequence, without its own scheduler, arbitrary DAG engine, or competing retry system. The toolkit owns artifact validity, compatibility, and authorization in both execution paths.

Transactional metadata records model state, policies, runs, experiments, and artifact references. Analytical data and immutable snapshots live outside that metadata store. The first implementation may use SQLite, but no generic storage-plugin framework is justified without a second demonstrated backend.

Completed stages have immutable resolved execution manifests and identifiable finalized artifacts. Reuse requires matching relevant inputs, definitions, parameters, policies affecting the stage, and contracts. Partial artifacts must never appear complete. Failure reports distinguish reused, invalidated, and rerun stages. Recovery does not imply exactly-once external API delivery.

## 12. Reports and agent behavior

Structured evaluation artifacts are authoritative. An agent creates a Marimo notebook from those artifacts, with charts, caveats, comparison results, and recommended next experiments. A report must not silently rerun against live source data or redefine metric calculations. The ML runtime must operate without Marimo installed in production.

The toolkit owns canonical version-matched instructions and task-oriented skills; supported agents use minimal non-destructive registration and must not maintain divergent copies of behavioral rules. A user should be able to adopt it in an existing repository without overwriting existing agent guidance. Minimal reversible registration must be verified in both Claude Code and Codex without overwriting existing project instructions. Agent compatibility claims require independent acceptance tests, not merely portable Markdown.

## 13. Required validation

**Always blocking:** unauthorized writes, missing required inputs, incompatible feature contracts, invalid artifacts, known violations of declared temporal availability, inconsistent model activation, and unauthorized publication.

**Policy-governed:** minimum predictive or business metric thresholds, sample requirements, freshness, production approval, and other application-specific gates.

**Warnings unless elevated by policy:** distribution drift, weak evidence, suspicious correlations, calibration deterioration, and unstable business results. A warning is not evidence of safety.

## 14. Component ownership

- **Core contracts:** definitions, compatibility rules, and validation semantics.
- **Data and features:** source access, profiling, ingestion boundary, SQL definitions, snapshots, and feature checks.
- **Models:** training adapters, imports, artifacts, versioning, activation, and inference compatibility.
- **Evaluation:** metric calculations, baselines, backtests, experiments, and authoritative results.
- **Execution and metadata:** run coordination, dependency checks, transactional state, and recorded outcomes.
- **Reporting and integrations:** Marimo, Dagster, external prediction destinations, and other optional adapters.
- **Agent guidance:** discoverable workflows and explanations; never authoritative enforcement logic.

These are ownership boundaries, not instructions to create one package or interface for every item. One repository and one initial Python distribution are preferred. Optional integrations may depend on core operations; core operations must not depend on them.

## 15. MVP acceptance

1. **New model:** An unfamiliar relational dataset is inspected, important ambiguities are clarified, reusable SQL features are created, an immutable snapshot is evaluated with a baseline and XGBoost candidate, and validated predictions are available to an application.
2. **Existing model:** A supplied compatible model can be imported, its unknown provenance disclosed, its inputs validated, and predictions produced without retraining.
3. **Time-dependent workflow:** Declared prediction-time and availability rules reject a constructed look-ahead violation. A reproducible chronological backtest runs when requested.
4. **Decision support:** A qualification policy can gate on a predictive metric, a business metric, or both. The agent explains the evidence and uncertainties without overriding mandatory checks.
5. **Reporting:** An agent-generated Marimo report reads persisted evaluation artifacts and does not recalculate authoritative metrics.
6. **Application integration:** A non-Python application consumes batch predictions without running Python in its web process.
7. **Safety and continuity:** Unauthorized writes, incompatible features, failed publication, and interrupted or concurrent activation produce traceable outcomes.
8. **Agent portability:** At least two distinct coding agents can follow the same reference workflow and contracts without creating competing implementations.
9. **Generalization:** Two unrelated tabular use cases, such as customer churn and demand forecasting, work through the same core contracts without adding domain-specific concepts to the framework.

## 16. Explicit non-goals

No neural-network infrastructure, distributed training, streaming feature store, generalized connector marketplace, custom scheduler, hosted SaaS, billing, multi-tenant control plane, universal model-serving service, automated trading execution, comprehensive AutoML, or guaranteed causal inference. Dagster is optional. Marimo is the default report experience, not a production runtime dependency.

## 17. Contract gate decisions — consolidated

| Gate | Confirmed MVP contract |
|---|---|
| G1 | CSV, Parquet, DuckDB, read-only PostgreSQL; XGBoost training and narrow trusted import only |
| G2A | Immutable feature definition revisions; pinned model compatibility; semantic review for changed meaning |
| G2B | Prediction time, event time, availability semantics, and label availability at simulated training cutoffs |
| G3 | Qualification plus explicit approval or narrowly scoped revocable preauthorization for activation/rollback |
| G4A | Append-only prediction history by default, configurable retention, semantically defined current access |
| G4B | Canonical query access and authorized PostgreSQL publisher; independent delivery state and safe retries |
| G5A | Staged artifact creation, artifact-specific verification, immutable finalization and fingerprint |
| G5B | Toolkit determines artifact compatibility; Dagster coordinates dependencies/recovery; minimal direct path |
| G6A | Chronological holdout and expanding walk-forward, preserving fold-level evidence |
| G6B | ML/business metrics, denominators, sample size, defensible uncertainty, recorded decision assumptions |
| G7 | Claude Code and Codex as tested agents using one version-matched canonical guidance source |
| G8 | Agent-generated Marimo notebooks pinned to immutable evaluation evidence; no authoritative recalculation |

**Revised Decision 6 supersedes the original:** Dagster-preferred orchestration with minimal direct execution. Dagster remains optional, not a core dependency.

## 18. Normative authority and failure contracts

**Editable definitions:** Version-controlled project files own feature, metric, and policy intent. **Resolved execution manifest:** Every consequential run pins the exact definitions (including uncommitted edits), input artifacts, parameters, applicable policy revisions, and relevant environment identifiers actually used, excluding credentials and sensitive raw data. **Evidence:** Only finalized, verified immutable artifacts are reusable. **Operational state:** SQLite records run status, active-version pointers, authorizations, experiments, and delivery attempts. **Reports:** Derived, evidence-bound views.

| Failure | Required behavior |
|---|---|
| Unauthorized source read or destination write | Block, disclose required permission; never escalate automatically |
| Ambiguous target, grain, or availability | Ask user before committing consequential assumptions |
| Feature revision mismatch | Block implicit model consumption and identify affected dependencies |
| Known temporal leakage | Block affected historical evaluation; do not claim general leakage-proof status |
| Interrupted artifact finalization | Never expose partial artifact as finalized; detect and reconcile on recovery |
| Changed required dependency | Refuse unsafe reuse and report mismatch; Dagster may schedule recomputation |
| Missing qualification evidence or authority | Block activation; preserve existing active model |
| Concurrent activation | Atomic conflict-safe transition; record winner and rejected attempt |
| Partial/unknown external publishing | Preserve canonical batch; record delivery state; retry only with verified safety |
| Report generation failure | Keep authoritative evaluation intact; report can be regenerated |

**Prediction semantics:** Stable identities across publishing retries; multiple horizons or valid times for the same entity must remain distinct. Current means applicable to the requested target and horizon, not simply most recently written. Authorized retention cleanup must identify any impact on referenced evaluation evidence. Publishing cannot implicitly modify the destination schema.

**Backtesting semantics:** Distinguish contemporaneously recorded from retrospectively reconstructed predictions; actions from abstentions; observed outcomes from hypothetical effects. Preserve each fold's training cutoff, evaluation period, model and feature revisions, sample counts, denominators, decision-policy revision, and costs. Do not invent uncertainty estimates where assumptions do not support them. Historical replay is not causal inference or proof of future value.

## 19. Acceptance tests for the specification

1. Unfamiliar PostgreSQL data is safely profiled, ambiguities are raised, SQL features and an immutable Parquet training snapshot are produced, and baseline/XGBoost are evaluated before canonical predictions are queried.
2. A supported existing XGBoost model with missing training history can be imported with explicit provenance limitations and must satisfy feature and qualification checks before activation.
3. A constructed late-arrival or unavailable-label case fails temporal validation; chronological and expanding walk-forward results preserve independently inspectable folds.
4. A business metric can gate activation while an ML metric remains a monitoring signal; missing required evidence blocks activation.
5. Concurrent activation cannot yield two active versions; rollback verifies compatibility and authorization.
6. Partial PostgreSQL publishing leaves inference valid and records partial/unknown delivery; safe retries do not change canonical prediction identities.
7. Interrupted artifact creation never produces reusable checkpoints; a compatible finalized artifact is reusable in both Dagster and direct paths with an explanation.
8. An agent-generated Marimo notebook reads pinned results, labels exploration separately, and cannot replace authoritative scores.
9. Claude Code and Codex discover version-matched skills in an existing repository without overwriting instructions and satisfy the same validation/failure rules.
10. Two unrelated tabular domains use the same contracts without domain-specific core changes.

## 20. Independent adversarial freeze review

**Verdict: Ready for specification freeze review, not yet implementation authorization.** All eight contract areas have product-level decisions. Remaining bounded precision tasks are: exact manifest fields/fingerprint algorithm and interrupted filesystem/SQLite reconciliation; trusted XGBoost artifact formats; minimum statistically defensible uncertainty estimators; current-prediction tie-breaking and retention consequences; PostgreSQL idempotency prerequisites; concrete Claude/Codex registration mechanics; and Dagster acceptance coverage. These must be stated as testable requirements before implementation tickets. None justifies a new subsystem.

**Anti-slop findings:** No second orchestrator, generic connector marketplace, generic model-adapter framework, metric calculations inside notebooks, automatic model activation by score alone, or speculative cloud abstraction. Every new boundary must have a real user scenario, one owner, explicit input/output/failure semantics, and no duplicate authority. No code, tickets, or scaffolding are authorized by this document.