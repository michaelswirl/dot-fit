# Agent operating contract — review draft v0.3

**Companion to MVP spec v0.6. Agent-neutral by design; compatibility requires testing.**

## Purpose
Agents investigate, propose, operate, interpret, and report. Deterministic toolkit operations enforce contracts and production permissions. A user should understand what was built without reading generated code.

## Canonical agent workflow
1. Inspect application environment and approved data sources using bounded, privacy-conscious discovery.
2. Explain observed facts, inferred meanings, confirmed facts, and consequential unknowns; ask about entity grain, targets, timestamps, and availability before modeling.
3. Propose a bounded plan: shared feature contracts, baseline, candidate, holdout evaluation, expected output, and resource needs.
4. Prefer DuckDB SQL; use explicit Python transforms only when justified. Do not invent alternate registries or evaluation systems.
5. Use toolkit operations for snapshots, training, evaluation, inference, and policy checks. Do not replace runtime checks with agent reasoning.
6. Explain ML and optional business metrics, uncertainty, limitations, and next experiments. Recommend backtests for time-dependent models without claiming they prove future value.
7. Generate Marimo reports from pinned evidence; never silently recalculate authoritative metrics or read live source data.
8. Seek authorization before activation, policy weakening, or external publishing unless an approved policy already grants it.
9. On failure, report reused, invalidated, and rerun stages with reasons.

## Authority table
| Operation | Default agent authority |
|---|---|
| Bounded source discovery | Autonomous within granted permissions and budget |
| Feature proposals, candidate training, evaluation | Autonomous within approved resource bounds |
| Define business metrics or policy thresholds | Propose; obtain confirmation for consequential assumptions |
| Production activation | Explicit approval or prior approved policy |
| External publication | Explicit approval or prior approved policy |
| Weaken production safeguards | Explicit approval |
| Override mandatory validation | Never |

## Skills and portability
Version canonical task-oriented skills with the toolkit. Cover data discovery, features, training/import, evaluation/backtesting, diagnosis, and publishing. Preserve existing repository instructions. Do not fork core behavior by agent provider. Prove compatibility with Claude Code and Codex; concrete registration mechanics remain to be verified during freeze review.

## Required explanations
What data was used; what is observed versus assumed; what each feature means; which model and version were evaluated; which metrics, baselines, periods and policies applied; why the model is or is not qualified; where canonical predictions are accessible; and what was authorized for external publishing.

## Honest failure behavior
Never invent training provenance, feature semantics, statistical certainty, or business returns. Report unsupported integrations and missing evidence explicitly. Do not treat successful execution as proof of predictive usefulness. No agent may self-authorize a failed production gate.

## Review questions
Can a second agent continue from recorded artifacts without reinterpreting undocumented assumptions? Can the user inspect decisions without agent-specific tools? Does every consequential side effect have an explicit authority record? If not, the workflow is incomplete.

## Resolved run and delivery behavior
Every consequential run uses a frozen resolved execution manifest rather than relying only on a Git revision. Agents must explain what definitions and evidence were actually used. On recovery, agents report reused, invalidated, rerun and refused stages; the toolkit makes the authoritative compatibility decision.

Canonical prediction success and external publication success are separate. Agents must not automatically retry an API write when delivery is uncertain and idempotency is unverified. They must never replace stable prediction identities during a delivery retry.

Historical backtests must distinguish observed predictions from retrospective reconstructions, actions from abstentions, and observed outcomes from simulated consequences. The agent must not present a backtest as proof of future performance.

## Skill discovery contract
Canonical skills are version-matched to the installed toolkit. Registration in consuming repositories must be minimal, non-destructive, and reversible. Agent-specific discovery may differ, but core rules cannot fork. Cross-agent claims require the same acceptance scenario to succeed on two distinct agents.

## Outstanding acceptance details
Product-level choices are resolved: evidence-bound Marimo, versioned policy authority, chronological/expanding backtests, and two reference agents. Concrete registration, evidence schema, and authorization representation require testable acceptance details before implementation approval.

## Consolidated agent acceptance behavior
Claude Code and Codex are the MVP acceptance agents. Both must discover package-owned, version-matched guidance through minimal reversible registration without overwriting existing project instructions. Other agents are not certified by the MVP.

Agents should recommend time-aware backtesting when appropriate; chronological holdout and expanding walk-forward are toolkit operations, not improvised notebooks. They must disclose unavailable uncertainty, fold instability, observed versus simulated outcomes, and whether business metrics are decision gates or only diagnostics.

Production actions require specific recorded authorization or narrowly scoped revocable preauthorization. Agents may propose metric thresholds and feature semantic revisions but may not silently assert compatibility, weaken policies, or promote a candidate. They must explain when a Dagster run succeeds but toolkit artifact checks fail.

For predictions, agents use canonical query access first. PostgreSQL publishing is optional, authorized, and independently tracked. A partial or unknown delivery cannot be described as success; automatic retries require verified idempotency. Agent-generated Marimo reports must pin immutable evidence and label exploratory computations separately from authoritative metrics.

## Remaining verification before freeze
Test actual Claude/Codex registration and stale-guidance handling; verify that agents respond correctly to rejected temporal features, incomplete artifacts, missing approvals, and unknown publishing outcomes. Do not invent an agent-specific alternate runtime.