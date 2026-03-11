# Training Bootstrap Specification

Related specs: [architecture](architecture.md), [runtime](runtime.md), [research synthesis](research-synthesis.md), [open questions](open-questions.md), [bibliography](bibliography.md).

## Scope
This document defines how the system is initialized and evolved from pre-runtime setup into live operation.

## Objective and completion criteria for this milestone
**Objective:** convert the bootstrap roadmap into an execution-ready checklist for simulator planning while preserving the only-L1-persists invariant.

**Completion criteria:**
- Each staged bootstrap step has explicit entry criteria, measurable exit criteria, required artifacts, and failure/rollback conditions.
- A live-start readiness gate is defined and must pass before runtime is marked live.
- External training proposals are constrained to auditable L1 delta injection semantics.

## Core rule
Initialization may use search-heavy methods (NEAT, brute-force probes, hyperparameter sweeps), but after live start the system updates only through persisted L1 state transitions.

- **Paper-backed component:** NEAT as a topology/bootstrap method (see [bibliography](bibliography.md#bootstrap--optimization--optional-appendices)).
- **Project synthesis:** hard boundary between bootstrap tooling and live-update regime.

## Bootstrap assumptions
- No claim that bootstrap search is sample-efficient.
- Goal is to discover minimally stable replicator dynamics and viable routing seeds.
- Bootstrap artifacts are converted into initial L1 snapshots before runtime start.

## Staged roadmap
### 1) Bootstrap stable replicators
Find local policies that maintain bounded activity and recover from perturbation.
- Candidate methods: NEAT seeds, brute-force policy sweeps.

Checklist:
- **Entry criteria (Project synthesis):** L1 schema version is frozen for the run; perturbation suite and recovery score thresholds are defined.
- **Measurable exit criteria (Project synthesis):** at least 95% recovery success over the agreed perturbation suite for 3 independent seeds, with no unresolved safety guardrail violation.
- **Required artifacts (Project synthesis):** seed manifest, perturbation suite definition, per-seed recovery traces, and selected bootstrap L1 snapshot set.
- **Failure/rollback conditions (Project synthesis):** if recovery success drops below threshold or safety violations persist, discard candidate snapshots and restart from previous accepted seed manifest.

### 2) Instantiate tiny experts
Introduce tiny sparse expert kernels per address family.
- Prefer parameter-efficient experts with clear compute caps.

Checklist:
- **Entry criteria (Project synthesis):** stable replicator snapshot set is accepted and compute budget per tick is fixed.
- **Measurable exit criteria (Project synthesis):** expected-k activation stays within ±10% of target on benchmark workloads, and expert fault injection remains isolated to local address families.
- **Required artifacts (Project synthesis):** expert family map, expected-k benchmark report, and fault-isolation test traces.
- **Failure/rollback conditions (Project synthesis):** if expected-k drift or cross-family failure spread exceeds thresholds, revert to previous accepted expert map and retune caps.

### 3) Introduce routing behavior
Add neighbor, explicit, and capability routing policies.
- Tune routing priors for load spreading and specialization.

Checklist:
- **Entry criteria (Project synthesis):** tiny expert benchmarks pass and routing observability fields from [runtime](runtime.md) are available in traces.
- **Measurable exit criteria (Project synthesis):** P95 routing latency and dropped-frame rate remain inside envelope targets for both sparse and bursty test regimes.
- **Required artifacts (Project synthesis):** routing policy config, collision/fairness scenario traces, and benchmark envelope summary.
- **Failure/rollback conditions (Project synthesis):** if bursty regime exceeds congestion envelope for 2 consecutive benchmark runs, revert routing priors to last accepted profile.

### 4) Add async signaling
Enable full reflex/deliberation plane interaction.
- Validate interrupt paths and escalation loops.

Checklist:
- **Entry criteria (Project synthesis):** routing envelope targets are met and rollback event schema is implemented in trace output.
- **Measurable exit criteria (Project synthesis):** interrupt-to-action latency meets plane budget, rollback rate is below threshold under burst ingress, and cooldown transitions have zero orphaned frames.
- **Required artifacts (Project synthesis):** reflex/deliberation interaction traces, escalation-loop timing report, rollback audit report.
- **Failure/rollback conditions (Project synthesis):** if rollback storms or orphaned frames appear, disable async escalation path and fall back to prior synchronous profile.

### 5) Add weight-space generation
Test generated-weight pathways for selected expert classes.
- **Paper-backed component:** informed by neural-manifold/weight-generation literature in [bibliography](bibliography.md#weight-space-generation--neural-manifold).
- **Open question:** stable canonicalization across generated parameter sets.

Checklist:
- **Entry criteria (Project synthesis):** async signaling is stable and selected expert classes have canonical baseline checkpoints.
- **Measurable exit criteria (Project synthesis):** generated weights meet or exceed baseline quality threshold with bounded rollback increase (<5% absolute) on target tasks.
- **Required artifacts (Project synthesis):** generator config, baseline-vs-generated evaluation report, and canonicalization diagnostics.
- **Failure/rollback conditions (Project synthesis):** if rollback increase or instability exceeds bounds, disable generated-weight pathway and continue with fixed expert weights.

### 6) Add modality executors
Attach few-step executor options for language or other modalities.
- Evaluate tradeoffs between speed, controllability, and trace clarity.
- Keep executor APIs interchangeable where possible.

Checklist:
- **Entry criteria (Project synthesis):** weight-generation policy decisions for candidate expert classes are frozen for the milestone.
- **Measurable exit criteria (Project synthesis):** each modality executor passes shared task suite with trace-clarity score and latency inside declared operating envelope.
- **Required artifacts (Project synthesis):** modality task suite definition, per-executor scorecards, and interchangeability compliance notes.
- **Failure/rollback conditions (Project synthesis):** if an executor fails envelope targets, it remains experimental-only and is excluded from live-start readiness.

## Live-start rule (post-bootstrap)
Once the runtime is marked live:
- No direct global reinitialization of hidden state.
- Durable adaptation occurs only as L1 commits.
- External training jobs may propose updates, but they must be injected as explicit L1 deltas.

## Live-start readiness gate
- **Project synthesis:** runtime may be marked live only if all gate checks pass in one evaluation window.

Gate checklist:
1. **Bootstrap artifact integrity:** all required artifacts from stages 1–6 are present, versioned, and linked to the accepted L1 snapshot set.
2. **Trace reproducibility:** minimum debug packets replay deterministically for acceptance scenarios from [tick trace](tick-trace.md).
3. **Safety envelope compliance:** rollback, cooldown, and throttle metrics remain inside declared envelopes for sparse and bursty ingress.
4. **Routing and scheduler stability:** no unresolved starvation violations and no unresolved dangling-handle cleanup failures.
5. **Only-L1-persists audit:** no bootstrap mechanism requires non-L1 durable runtime state.

Failure behavior:
- **Project synthesis:** if any gate check fails, runtime remains pre-live and must use the previous accepted L1 snapshot baseline.

## External training proposal injection policy
- **Project synthesis:** external/offline training outputs are advisory and cannot mutate runtime state directly.

Required proposal packet:
1. target address set and schema version,
2. candidate L1 delta payload,
3. provenance metadata (origin run, data slice, evaluation context),
4. expected metric deltas and risk notes,
5. rollback plan if accepted delta regresses behavior.

Admission and commit semantics:
- **Project synthesis:** proposals enter through the same validation pipeline as internal deltas (schema validation, safety guardrail checks, changed-path audit).
- **Project synthesis:** accepted proposals commit only at normal commit boundaries; rejected proposals are logged with reasons for reproducibility.
- **Open question:** should proposal admission include a temporary quarantine tier before full commit under high-risk modalities? Track in [open questions](open-questions.md).

## Evaluation guidance
Track at least:
- stability under load,
- rollback frequency,
- routing efficiency,
- expert utilization sparsity,
- task-level quality metrics per modality.

## Optional appendices
Future appendices may compare local-learning and optimizer choices (e.g., Mono-Forward, Muon-like optimizers) for offline bootstrap experiments. These are optional supports, not mandatory architecture commitments.

## Clarification
NEAT is a **bootstrap method**, not the long-term learning method for ongoing system adaptation.
