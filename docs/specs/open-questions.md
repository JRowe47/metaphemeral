# Open Questions

Related specs: [architecture](architecture.md), [runtime](runtime.md), [activation store](activation-store.md), [training bootstrap](training-bootstrap.md), [glossary](glossary.md), [bibliography](bibliography.md).

This file tracks unresolved design choices and research risks. Items here are intentionally non-final.

## 1) Exact L1 state representation
- Should L1 store full tiny weights, latent codes, or hybrid compressed artifacts?
- What schema best supports reproducibility and partial rollback?
- How should versioning be encoded for long-lived addresses?

## 2) Canonicalization for weight-space generation
- How do we canonicalize generated weights so semantically equivalent experts map to stable representations?
- Should canonicalization be task-conditioned or architecture-conditioned?
- What is the tolerance threshold for "equivalent" experts in routing decisions?

## 3) Executor template choice
- When should few-step flow-based execution be preferred over block diffusion editing?
- Is a unified abstract executor interface sufficient, or do we need modality-specific wrappers?
- What observability primitives are needed to compare executor behavior fairly?

## 4) Routing semantics at scale
- How should capability routing resolve collisions under high fan-in?
- Should capability beacons decay continuously or in stepwise epochs?
- Do we need fairness constraints to prevent expert starvation?

## 5) Activation-store sizing and overflow
- What capacity per address is needed for realistic workloads?
- Which overflow objective is best: preserve recency, preserve priority, or preserve dependency completeness?
- How should overflow policies differ between reflex and deliberation traffic?

## 6) Global baseline activity control
- What minimal global control signals are necessary to prevent runaway activation?
- Can control remain mostly local while still achieving stable long-range coordination?
- How should baseline activity targets adapt across modalities?

## 7) Multimodal codec strategy
- Should modalities share one latent codec family or use modality-specific codecs with a shared adapter?
- How do codec errors propagate through handle-based activation references?
- What compression level is acceptable before routing quality degrades?

## 8) Dormancy defaults across the lattice
- What fraction of addresses should be dormant by default?
- How should dormant nodes advertise capability without causing routing noise?
- What wake-up policy balances latency and efficiency?

## 9) Operational meaning of "reasoning depth"
- Is reasoning depth measured by deliberation steps, trace branching, or commit impact?
- Should depth budgets be absolute per tick or adaptive by pressure class?
- How should we detect unproductive depth versus useful exploration?

## Near-term validation priorities
1. Freeze a provisional L1 schema and run simulator traces.
2. Stress-test activation-store overflow behavior.
3. Benchmark routing stability under sparse and bursty traffic.
4. Compare executor options on traceability and latency.
