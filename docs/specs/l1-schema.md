# L1 Schema Specification (Provisional v0)

Related specs: [architecture](architecture.md), [runtime](runtime.md), [activation store](activation-store.md), [tick trace](tick-trace.md), [open questions](open-questions.md), [glossary](glossary.md).

## Scope and status
This document defines a **provisional** per-address L1 schema for simulator planning. It is intentionally minimal and versioned.

- **Project synthesis:** the concrete field layout and commit rules in this spec.
- **Open question:** binary encoding and compression choices for long-lived deployments.
- **Constraint:** preserve the only-L1-persists invariant from the architecture spec.

## Objectives for v0
1. Make L1 durable state explicit enough for deterministic simulation.
2. Keep field count small so update/rollback semantics stay auditable.
3. Preserve compatibility with generated L2/L3 routines without persisting them.

## L1 envelope
Each address persists one `L1Record`:

```yaml
L1Record:
  schema_version: "l1.v0"
  address_id: "hex://q/r/s"
  lifecycle:
    created_tick: 0
    last_commit_tick: 0
    commit_counter: 0
    status: active # active | quarantined | dormant

  adaptation_seed:
    latent_seed_ref: "seed://..."
    seed_hash: "sha256:..."
    generator_family: "wg-baseline"

  routing_profile:
    capability_tags: ["cap.translate", "cap.safety.filter"]
    neighbor_bias: 0.65
    explicit_affinity:
      - target: "hex://0/1/-1"
        weight: 0.8
    capability_affinity:
      - capability: "cap.translate"
        weight: 0.9

  safety_state:
    cooldown_until_tick: 0
    violation_score: 0.0
    quarantine_level: 0

  provenance:
    lineage_id: "lineage://..."
    parent_commit: "commit://..."
    trace_summary_hash: "sha256:..."
    last_update_reason: "routing-adjustment"
```

## Field requirements
### 1) Identity and versioning fields
- `schema_version` must be validated before commit merge.
- `address_id` is immutable after bootstrap.
- `lifecycle.commit_counter` increments exactly once per accepted commit.

### 2) Adapter / latent seed container
- `adaptation_seed` is durable but opaque to runtime scheduling.
- `latent_seed_ref` points to external artifact storage; L1 keeps reference + integrity hash only.
- Runtime may instantiate L2/L3 behaviors from this seed and current activation context, but generated routines are transient.

### 3) Routing priors and capability descriptors
- `routing_profile.capability_tags` drives capability-beacon publication.
- `neighbor_bias` is a bounded scalar in `[0,1]` used by local routing policy.
- Affinity lists are optional and can be empty; absent values imply neutral priors.

### 4) Safety and cooldown metadata
- `cooldown_until_tick` gates reflex escalation retries.
- `violation_score` accumulates recent policy pressure and decays by runtime rule.
- `quarantine_level` controls allowed emission/routing modes.

### 5) Lineage / provenance fields
- `lineage_id` binds descendants across updates.
- `parent_commit` and `trace_summary_hash` enable deterministic replay checks.
- `last_update_reason` is a short string enum (implementation-defined list in simulator).

## Commit and rollback semantics (field-level)
### Commit preconditions
A candidate delta can be committed only if all checks pass:
1. `schema_version` matches supported parser.
2. Every changed field is in the allowlist for the originating plane.
3. `trace_summary_hash` is present for any non-noop update.
4. Safety guardrails (`quarantine_level`, violation caps) are not breached.

### Allowed writes by plane
- **Reflex plane:** `safety_state.*`, limited `routing_profile.neighbor_bias` adjustments, provenance updates.
- **Deliberation plane:** `routing_profile.*`, `adaptation_seed.*` reference rotation, provenance updates.
- **Both planes:** `lifecycle.last_commit_tick`, `lifecycle.commit_counter` (via commit service only).

### Rollback behavior
On rollback:
- Restore entire prior `L1Record` snapshot.
- Append failure reason to non-durable trace log (not in L1).
- Do **not** increment `commit_counter`.

## Version migration policy
- New schemas must be additive-first (`l1.v0` -> `l1.v1` adds optional fields).
- Breaking removals require explicit migrator and replay parity tests.
- Mixed-version neighborhood operation is allowed if capability publication can be downcast to shared fields.

## Failure modes (v0)
- Missing `trace_summary_hash` causes commit rejection.
- Unsupported `schema_version` causes quarantine-level escalation path.
- Stale `latent_seed_ref` with hash mismatch causes adaptation-seed update rejection.

## Open questions and validation path
- **Open question:** should `adaptation_seed` allow multiple concurrent generator families?
  - Validation path: compare replay stability vs single-family baseline in simulator scenarios.
- **Open question:** should `violation_score` be fixed-point or float for deterministic cross-platform replay?
  - Validation path: run deterministic replay suite across two runtimes and compare commit hashes.
