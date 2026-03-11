# Canonical Tick Trace (v0)

Related specs: [runtime](runtime.md), [l1 schema](l1-schema.md), [activation store](activation-store.md), [architecture](architecture.md), [glossary](glossary.md).

## Purpose
This document defines one canonical one-tick trace for simulator planning, from ingress to commit/rollback.

- **Project synthesis:** trace event contract and field-level expectations.
- **Open question:** minimum payload size required for long-term observability.
- **Constraint:** only L1 is durable; frames, packets, and generated routines are transient.

## Tick trace record shape
```yaml
tick_trace:
  tick_id: 1042
  address_id: "hex://0/0/0"
  ingress:
    packet_count: 2
    trace_ids: ["tr-8a", "tr-8b"]
  pressure_snapshot:
    ready_frames: 5
    blocked_frames: 1
    overflow_risk: 0.32
  schedule:
    reflex_dispatched: ["h-101", "h-102"]
    deliberation_dispatched: ["h-103"]
  emissions:
    urgent_packets: 1
    normal_packets: 2
  l1_delta:
    changed_paths:
      - "routing_profile.neighbor_bias"
      - "safety_state.cooldown_until_tick"
      - "provenance.trace_summary_hash"
  commit:
    status: accepted # accepted | rolled_back
    reason: "guardrails_passed"
```

## Required per-tick trace metadata (simulator contract)
- **Project synthesis:** the following metadata is required for replayability in the v0 simulator.

Each tick trace must include:
1. **Identity and lineage**
   - `tick_id`, `address_id`, `trace_ids[]`, `parent_trace_ids[]`.
2. **Pressure and capacity snapshot**
   - `ready_frames`, `blocked_frames`, `overflow_risk`, `store_fill_ratio`.
3. **Scheduling decisions**
   - `reflex_dispatched[]`, `deliberation_dispatched[]`, `deferred_handles[]`.
4. **Routing and emissions summary**
   - `urgent_packets`, `normal_packets`, `dropped_packets`, `drop_reasons[]`.
5. **Durable delta intent**
   - `changed_paths[]`, `candidate_delta_hash`, `validator_version`.
6. **Outcome**
   - `commit.status`, `commit.reason`, `rollback_event_id` (nullable).

Failure mode note:
- If any required field is missing, the trace is flagged `trace_incomplete=true` and is not eligible as a gold replay artifact.

## Canonical one-tick walkthrough
1. **Ingress capture:** two packets arrive and are normalized into activation frames.
2. **Store update:** frames `h-101`, `h-102`, `h-103` become ready; pressure metrics are sampled.
3. **Reflex dispatch:** `h-101` handles high-priority safety filter, `h-102` handles urgent route normalization.
4. **Deliberation dispatch:** `h-103` proposes routing-prior adjustment based on observed congestion.
5. **Emission:** one urgent packet (quarantine advisory) and two normal packets are sent.
6. **Commit validation:** candidate L1 delta updates routing and safety cooldown plus trace hash.
7. **Commit result:** accepted and persisted as next L1 snapshot for this address.

## L1 fields touched in canonical trace
From [l1-schema.md](l1-schema.md):
- `routing_profile.neighbor_bias` adjusted from `0.65 -> 0.62`.
- `safety_state.cooldown_until_tick` adjusted from `1040 -> 1044`.
- `provenance.trace_summary_hash` replaced with current trace hash.

No other durable fields change in this canonical tick.

## Rollback variant (guardrail failure)
If deliberation proposes `neighbor_bias = 1.4` (out of bounds):
- validation fails,
- `commit.status` becomes `rolled_back`,
- all L1 fields remain as pre-tick snapshot,
- trace is marked for reflex retry policy.

## Minimum debug packet (v0 proposal)
Each emitted debug packet should contain:
- `tick_id`,
- `address_id`,
- `trace_id`,
- `dispatch_plane`,
- `changed_paths` (if any),
- `commit_status`,
- `rollback_event_id` (nullable),
- `packet_seq` (monotonic per tick).

Validation rule:
- A replay attempt must fail fast if two debug packets in the same tick share the same `packet_seq`.

## Rollback event schema (required)
- **Project synthesis:** rollback events are transient runtime records; they are not durable state unless summarized through existing L1 provenance fields.

```yaml
rollback_event:
  rollback_event_id: "rb-1042-01"
  tick_id: 1042
  address_id: "hex://0/0/0"
  trace_id: "tr-8b"
  reason_code: "guardrail_violation" # validation_failed | guardrail_violation | dependency_gap | consistency_break
  violated_guardrail: "routing_profile.neighbor_bias <= 1.0"
  dependency_gap:
    missing_handles: ["h-201"]
    missing_trace_ids: ["tr-7f"]
  attempted_changed_paths:
    - "routing_profile.neighbor_bias"
  scheduler_action: "retry_reflex_then_defer_deliberation"
```

Pass/fail behavior for simulator testing:
- Pass: rollback event contains non-empty `reason_code`, `trace_id`, and either `violated_guardrail` or non-empty `dependency_gap`.
- Fail: rollback event is emitted without enough detail to reproduce why commit validation failed.

## Open question and validation path
- **Open question:** should `pressure_snapshot` be persisted in L1 for forensic replay shortcuts?
  - Validation path: benchmark replay speed gains against increased durable-state cost and only-L1-persists strictness.
