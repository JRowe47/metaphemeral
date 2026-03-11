# Training Bootstrap Specification

Related specs: [architecture](architecture.md), [runtime](runtime.md), [research synthesis](research-synthesis.md), [open questions](open-questions.md), [bibliography](bibliography.md).

## Scope
This document defines how the system is initialized and evolved from pre-runtime setup into live operation.

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
- Exit criterion: reproducible recovery behavior over stress scenarios.

### 2) Instantiate tiny experts
Introduce tiny sparse expert kernels per address family.
- Prefer parameter-efficient experts with clear compute caps.
- Validate sparse activation patterns and failure isolation.

### 3) Introduce routing behavior
Add neighbor, explicit, and capability routing policies.
- Tune routing priors for load spreading and specialization.
- Measure congestion, latency, and dropped-frame rates.

### 4) Add async signaling
Enable full reflex/deliberation plane interaction.
- Validate interrupt paths and escalation loops.
- Confirm rollback and cooldown logic under bursty ingress.

### 5) Add weight-space generation
Test generated-weight pathways for selected expert classes.
- **Paper-backed component:** informed by neural-manifold/weight-generation literature in [bibliography](bibliography.md#weight-space-generation--neural-manifold).
- **Open question:** stable canonicalization across generated parameter sets.

### 6) Add modality executors
Attach few-step executor options for language or other modalities.
- Evaluate tradeoffs between speed, controllability, and trace clarity.
- Keep executor APIs interchangeable where possible.

## Live-start rule (post-bootstrap)
Once the runtime is marked live:
- No direct global reinitialization of hidden state.
- Durable adaptation occurs only as L1 commits.
- External training jobs may propose updates, but they must be injected as explicit L1 deltas.

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
