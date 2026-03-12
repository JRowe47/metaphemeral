# Metaphemeral Network — Project To-Do

Related docs: [README](README.md), [Architecture](docs/specs/architecture.md), [Runtime](docs/specs/runtime.md), [Activation Store](docs/specs/activation-store.md), [Training Bootstrap](docs/specs/training-bootstrap.md), [Research Synthesis](docs/specs/research-synthesis.md), [Open Questions](docs/specs/open-questions.md), [Glossary](docs/specs/glossary.md), [Bibliography](docs/specs/bibliography.md).

## 1) Objective and completion criteria

**Objective:** turn current documentation into an execution-ready, reviewable task backlog for simulator planning while preserving the only-L1-persists architecture constraint.

**Completion criteria for this backlog (done when all are true):**
- [ ] A provisional L1 schema is frozen and used in at least one end-to-end tick trace design.
- [ ] Activation-store overflow policy is specified with measurable pass/fail behavior.
- [ ] Routing behavior is benchmark-defined for sparse and bursty regimes.
- [ ] Executor comparison plan (flow vs block diffusion) is defined with shared metrics.
- [ ] Open questions have owners, validation paths, and decision checkpoints.
- [ ] Cross-doc terminology is consistent with glossary definitions.

---

## 2) Non-negotiable constraints (from specs)

- [ ] Preserve **only-L1-persists** as the default system invariant unless explicitly revised by architecture decision.
- [ ] Keep distinctions explicit between **Paper-backed component**, **Project synthesis**, and **Open question** in any new spec text.
- [ ] Avoid implying implementation completeness; this remains documentation-first and pre-implementation.
- [ ] Keep runtime definitions concrete enough to support simulator design without hidden assumptions.

---

## 3) Priority roadmap

## P0 — Immediate (unblock simulator specification)

### A. Freeze provisional L1 state representation
- [x] Draft `docs/specs/l1-schema.md` with a concrete minimal schema:
  - [x] identity/versioning fields,
  - [x] adapter/latent seed container,
  - [x] routing priors and capability descriptors,
  - [x] safety and cooldown metadata,
  - [x] lineage/provenance fields for reproducibility.
- [x] Define commit/rollback field-level semantics for L1 updates.
- [x] Specify backward-compatible version migration policy for long-lived addresses.
- [x] Record unresolved parts as Open questions with explicit validation plan.

### B. Specify activation-store capacity and overflow behavior
- [x] Convert overflow policy into a deterministic rule set with tie-break order.
- [x] Define recommended per-address capacity bands by workload class (reflex-heavy vs deliberation-heavy).
- [x] Specify eviction objective tradeoff mode(s): recency, priority, dependency completeness.
- [x] Add overflow stress scenarios and expected outcomes to docs.
- [x] Define dangling-handle prevention/cleanup policy under aggressive clear/evict operations.

### C. Lock routing semantics under load
- [x] Define capability routing collision-resolution semantics (top-k, weighted random, or hybrid).
- [x] Define fairness and starvation guardrails for repeated high fan-in.
- [x] Define beacon refresh/decay rules and their observability fields.
- [x] Add routing policy examples for neighbor vs explicit vs capability traffic.
- [x] Specify admission/throttle logic when token buckets are depleted.

### D. Establish baseline tick-level observability contract
- [x] Define required trace metadata per tick (ids, pressure snapshots, schedule decisions, outcome).
- [x] Define “minimum debug packet” content for reproducible replay.
- [x] Define rollback event schema (reason, violated guardrail, dependency gap details).
- [x] Add a canonical one-tick trace example spanning mailbox → store → scheduler → commit.

### E. Decide first-pass executor evaluation protocol
- [x] Define shared benchmark tasks for flow-based and block-diffusion executors.
- [x] Define mandatory metrics: latency, trace clarity, correction stability, rollback rate.
- [x] Define acceptable operating envelopes by plane (reflex/deliberation budget constraints).
- [x] Record decision rule for when one executor is preferred over the other.

---

## P1 — Near-term (bootstrap and integration readiness)

### F. Convert training-bootstrap roadmap into milestone checklist
- [x] For each staged step (replicators → tiny experts → routing → async signaling → weight generation → modality executors), add:
  - [x] entry criteria,
  - [x] measurable exit criteria,
  - [x] required artifacts,
  - [x] failure/rollback conditions.
- [x] Define “live-start readiness” gate checklist before marking runtime live.
- [x] Define policy for external training proposals injected as L1 deltas.

### G. Turn open questions into decision records
- [x] Create a decision-log section or ADR-style folder for major unresolved questions.
- [x] For each open question, assign:
  - [x] owner,
  - [x] evidence required,
  - [x] validation method,
  - [x] deadline/checkpoint,
  - [x] fallback decision if evidence is inconclusive.
- [x] Start with the 4 near-term validation priorities from `open-questions.md`.

### H. Formalize safety and rollback policy variants
- [x] Define whether partial-accept rollback mode will exist in v1 simulator semantics.
- [x] Document guardrail thresholds that force rollback vs defer vs quarantine.
- [x] Specify reflex escalation actions for repeated failed deliberation traces.
- [x] Add examples of safe degradation under overload.

### I. Make glossary enforcement operational
- [x] Add “term introduction checklist” to spec contribution workflow.
- [x] Ensure new terms are entered in glossary with concise, implementation-oriented definitions.
- [x] Normalize terminology where docs currently use variants (e.g., generated routine/executor wording).

---

## P2 — Medium-term (research integration and policy hardening)

### J. Research-synthesis adoption matrix
- [x] Create a matrix mapping each imported paper-backed component to:
  - [x] adopted now,
  - [x] deferred,
  - [x] rejected,
  - [x] required preconditions,
  - [x] risks.
- [x] For each adopted item, define minimal simulator-facing interface assumptions.
- [x] For each deferred item, define re-evaluation trigger.

### K. Codec and multimodal policy definition
- [x] Decide shared codec family vs modality-specific codecs + shared adapter.
- [ ] Define acceptable compression/quality envelope before routing degradation.
- [ ] Define error-propagation handling for handle-referenced payload compression.

### L. Global activity and dormancy governance
- [ ] Define minimal global control signals needed to prevent runaway activation.
- [ ] Define default dormant fraction and wake-up policy.
- [ ] Define capability advertisement behavior for dormant nodes to reduce routing noise.

### M. Reasoning-depth and adaptive-compute semantics
- [ ] Define operational metric(s) for “reasoning depth.”
- [ ] Define per-plane depth budget policy (fixed vs pressure-adaptive).
- [ ] Define “unproductive depth” detection and abort/escalation behavior.

---

## 4) Cross-document maintenance tasks

- [ ] Add or update internal relative links whenever new spec pages are introduced.
- [ ] Keep README architecture summary aligned with spec changes.
- [ ] If any literature-backed claim changes, update `docs/specs/bibliography.md` in the same change.
- [ ] Ensure every substantive architecture/runtime claim stays labeled by classification policy.

---

## 5) Suggested execution order (first 3 work packages)

### Work package 1: L1 schema + tick trace backbone
- [x] Produce provisional L1 schema doc.
- [x] Add tick trace example that consumes/updates schema fields.
- [x] Verify compatibility with rollback and commit rules.

### Work package 2: Activation overflow + routing load semantics
- [x] Finalize overflow rule precedence and capacity recommendations.
- [x] Finalize capability collision and fairness semantics.
- [x] Add stress-test scenario definitions and expected outcomes.

### Work package 3: Executor comparison protocol
- [x] Define shared tasks/metrics.
- [x] Define decision thresholds and deployment recommendations.
- [x] Capture unresolved tradeoffs as Open questions with checkpoints.

---

## 6) Definition of done for each future spec PR

- [ ] Objective and measurable completion criteria stated in PR.
- [ ] Scope limited to required files; no incidental refactors.
- [ ] Relevant checks/manual validations listed.
- [ ] Risks and unvalidated assumptions explicitly called out.
- [ ] New unresolved decisions recorded as Open questions with validation path.
