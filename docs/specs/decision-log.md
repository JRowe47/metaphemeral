# Decision Log

Related specs: [open questions](open-questions.md), [runtime](runtime.md), [l1 schema](l1-schema.md), [activation store](activation-store.md), [tick trace](tick-trace.md), [training bootstrap](training-bootstrap.md), [glossary](glossary.md), [bibliography](bibliography.md).

This log tracks unresolved architecture/runtime choices as explicit decision records so simulator planning can proceed without hidden assumptions.

- **Project synthesis:** ADR-style decision tracking and checkpoint workflow in this repository.
- **Open question:** evidence from first simulator experiments is still pending for all records below.

## Record format (required)

Use one record per unresolved decision.

1. **ID / title**
2. **Status** (`proposed`, `in-validation`, `decided`, `deferred`)
3. **Classification** (`Paper-backed component`, `Project synthesis`, `Open question`)
4. **Owner** (single accountable role)
5. **Evidence required** (artifacts and quantitative thresholds)
6. **Validation method** (benchmark, replay audit, stress scenario, etc.)
7. **Checkpoint** (date or milestone trigger)
8. **Fallback if inconclusive** (safe default for simulator v1)
9. **Decision output** (filled when resolved)

## Active records (near-term priorities)

### DLR-001 — Provisional L1 schema freeze for replayable traces
- **Status:** `in-validation`
- **Classification:** Open question
- **Owner:** Architecture lead
- **Evidence required:**
  - At least 3 reproducible one-tick traces using the current L1 schema envelope.
  - Zero schema-field ambiguity findings in trace review.
  - Migration dry run between two schema versions with no data loss in required fields.
- **Validation method:** Run tick-trace replay checks and schema migration walkthrough against [l1 schema](l1-schema.md) and [tick trace](tick-trace.md).
- **Checkpoint:** Milestone `P1-G` review + first simulator trace bundle.
- **Fallback if inconclusive:** Freeze current schema as `v0-provisional`, forbid additive non-optional fields, and require adapter shims for any experimental fields.
- **Decision output:** _pending_

### DLR-002 — Activation-store overflow policy acceptance
- **Status:** `in-validation`
- **Classification:** Open question
- **Owner:** Runtime/scheduler lead
- **Evidence required:**
  - Stress outcomes for sparse, bursty, and dependency-heavy traffic classes.
  - Measured eviction correctness >= 99% against deterministic tie-break expectations.
  - No unresolved dangling-handle incidents after cleanup rules execute.
- **Validation method:** Execute overflow stress scenarios and rollback audits defined in [activation store](activation-store.md).
- **Checkpoint:** Milestone `P1-G` review + overflow test report artifact.
- **Fallback if inconclusive:** Use dependency-completeness-first eviction for deliberation and priority-recency hybrid for reflex, with stricter admission throttle.
- **Decision output:** _pending_

### DLR-003 — Routing stability under sparse vs bursty load
- **Status:** `in-validation`
- **Classification:** Open question
- **Owner:** Routing/policy lead
- **Evidence required:**
  - Collision resolution behavior captured for low, medium, and high fan-in traffic.
  - Starvation incidence below agreed threshold (`<0.5%` sustained-capability nodes over benchmark window).
  - Beacon refresh/decay observability fields populated in all benchmark traces.
- **Validation method:** Run routing benchmark suite and fairness audit defined in [runtime](runtime.md) and [tick trace](tick-trace.md).
- **Checkpoint:** Milestone `P1-G` review + routing benchmark artifact bundle.
- **Fallback if inconclusive:** Keep hybrid top-k + weighted-random routing with conservative fairness boosts and shorter beacon TTL.
- **Decision output:** _pending_

### DLR-004 — Executor protocol selection (flow vs block diffusion)
- **Status:** `in-validation`
- **Classification:** Open question
- **Owner:** Execution-research lead
- **Evidence required:**
  - Shared benchmark report with latency, trace clarity, correction stability, and rollback rate.
  - Tie-margin analysis against `epsilon_tie` and per-plane operating envelopes.
  - Reproducible minimum debug packets for both executor types.
- **Validation method:** Apply the runtime comparison protocol in [runtime](runtime.md) and [training bootstrap](training-bootstrap.md).
- **Checkpoint:** Milestone `P1-G` review + first executor comparison report.
- **Fallback if inconclusive:** Ship simulator v1 with both executors behind feature flags and route by task profile.
- **Decision output:** _pending_

## Maintenance policy

- Move a record to `decided` only after attaching the evidence artifact path and explicit threshold pass/fail call.
- Keep unresolved records in this file; do not bury unresolved items in narrative sections of other specs.
- When a record changes status, add a one-line note to [open questions](open-questions.md) and update related glossary terms if vocabulary changes.
