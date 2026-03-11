# Activation Store Specification

Related specs: [architecture](architecture.md), [runtime](runtime.md), [open questions](open-questions.md), [glossary](glossary.md).

## Purpose
This document specifies an explicit activation caching model for runtime computation. It is **not** a lossy summary vector and should not be treated as one.

- **Project synthesis:** activation stack store design in this repo.
- Inspired by event/modulation and online adaptation literature, but not copied from one paper.

## Mailbox vs activation stack store
### Mailbox
- Lightweight ingress queue for newly arrived packets.
- Minimal normalization and ordering.
- Short retention; no deep frame semantics.

### Activation stack store
- Structured frame store keyed by handles.
- Supports dependency tracking, pressure accounting, and frame lineage.
- Used by scheduler and experts as the explicit intermediate-state substrate.

Mailbox is for intake; activation stack store is for execution state.

## Activation frame schema
Each activation frame should include:
- `handle_id`: unique local reference.
- `trace_id`: execution lineage identifier.
- `origin`: source address/port.
- `route_mode`: neighbor | explicit | capability.
- `capability_tags`: optional target semantics.
- `priority`: scheduler hint.
- `ttl`: decay/expiry bound.
- `payload_ref`: pointer or compact payload block.
- `dependencies`: parent handles or required outputs.
- `created_tick` / `updated_tick`: timing metadata.
- `state`: ready | blocked | running | completed | dropped.

**Open question:** exact binary encoding for payload and dependency lists.

## Core operations
### Push
Insert a new frame from mailbox or local emission.
- Validate schema and bounds.
- Assign handle if missing.
- Update pressure counters.

### Pop
Remove and return a schedulable frame.
- Plane-aware prioritization (reflex may preempt).
- Honors dependency readiness.

### Peek
Inspect candidate frames without removal.
- Used for arbitration and diagnostics.

### Clear
Drop frames by policy scope.
- By trace id, expiry, overload class, or completed status.

## Handle-based messaging
Packets can carry handle references rather than full payloads when locality allows.
Benefits:
- lower packet size,
- explicit shared context across expert steps,
- easier traceability of multi-hop workflows.

Costs:
- requires handle lifetime management,
- potential dangling references under aggressive clearing.

## Pressure-based scheduling
Scheduler consumes pressure metrics emitted by the store:
- queue depth per priority band,
- age distribution,
- blocked/ready ratio,
- overflow risk score.

Pressure informs:
- reflex preemption thresholds,
- deliberation budget shrink/expand,
- routing throttles.

## Local backpressure and token-bucket style send limits
Each address maintains outbound token buckets by route class:
- neighbor bucket,
- explicit-address bucket,
- capability-routing bucket.

Send policy:
- consume token per packet (weighted by size/priority),
- replenish per tick at configured rates,
- trigger local backpressure when buckets deplete.

Backpressure actions:
- delay low-priority sends,
- compress or coalesce frames,
- escalate overflow risk to reflex plane.

## Overflow policy
On approaching or exceeding store limits:
1. Run `expiry_sweep` and drop all frames where `ttl <= current_tick`.
2. If usage is still above `soft_limit`, run deterministic eviction by configured objective mode.
3. If usage is still above `hard_limit`, demote non-critical deliberation frames to `blocked` with `overflow_backoff_until_tick`.
4. If usage is still above `hard_limit` after one demotion pass, emit an `overflow_signal` packet to reflex.
5. If usage stays above `hard_limit` for `quarantine_window_ticks`, trigger reflex quarantine mode.

- **Project synthesis:** this precedence order is the simulator default policy.

### Deterministic eviction objective modes
The store MUST use one configured mode per address class for predictable replay.

1. `recency_preserve` (default for reflex-heavy workloads)
   - Score: `(is_reflex_critical, -updated_tick, -priority, dependency_fanout)`.
   - Evict ascending score (lowest first).
2. `priority_preserve` (default for mixed workloads)
   - Score: `(is_reflex_critical, priority, -updated_tick, dependency_fanout)`.
   - Evict ascending score.
3. `dependency_preserve` (default for deliberation-heavy workloads)
   - Score: `(is_reflex_critical, dependency_fanout, priority, -updated_tick)`.
   - Evict ascending score.

Where:
- `is_reflex_critical` is `1` for reflex/safety frames, `0` otherwise.
- `dependency_fanout` is the number of children that directly depend on this frame.

### Tie-break order (all modes)
If multiple frames have identical score tuples, break ties in this exact order:
1. older `created_tick` first,
2. lexicographically smaller `trace_id` first,
3. lexicographically smaller `handle_id` first.

This tie-break order is required for deterministic simulator replay.

### Recommended capacity bands by workload class
- **Project synthesis:** per-address defaults for simulator baselines.

| Workload class | `soft_limit` (frames) | `hard_limit` (frames) | Default mode |
| --- | ---: | ---: | --- |
| reflex-heavy | 96 | 128 | `recency_preserve` |
| balanced | 192 | 256 | `priority_preserve` |
| deliberation-heavy | 384 | 512 | `dependency_preserve` |

Constraint: `hard_limit` MUST be at least `1.25 * soft_limit`.

### Dangling-handle prevention and cleanup
To prevent stale `payload_ref` and dependency references:
1. Every eviction writes a `tombstone` record for `handle_id` with `evicted_tick` and `trace_id`.
2. Any frame listing a tombstoned handle in `dependencies` is marked `blocked_missing_dependency`.
3. `blocked_missing_dependency` frames are retried for `retry_window_ticks`.
4. If dependency is not restored by retry expiry, frame is dropped with reason `dependency_timeout`.

- **Project synthesis:** tombstones are local-only metadata and are not persisted as L1 state.

### Overflow stress scenarios (simulator acceptance tests)
1. **Burst ingress, reflex-priority protection**
   - Setup: start at 80% of `soft_limit`; inject burst to 140% of `hard_limit` with mixed priorities.
   - Expected: all expired frames removed first, reflex-critical frames survive first eviction pass, overflow signal emitted.
2. **Dependency fanout preservation under deliberation load**
   - Setup: `dependency_preserve` mode with DAG-shaped traces and 20% random TTL expiry.
   - Expected: high-fanout parents survive longer than leaf frames; dependency-timeout drops are logged.
3. **Aggressive clear during backlog**
   - Setup: invoke `clear(trace_id)` while store is above `soft_limit`.
   - Expected: tombstones generated, dependent frames move to `blocked_missing_dependency`, no unresolved handle references remain after retry window.

## Why this is not a simple vector summary
A vector summary collapses temporal and dependency structure into one representation. The activation stack store retains explicit, queryable, and schedulable units with lineage and control metadata. This supports fine-grained rollback, debugging, and pressure-aware scheduling that would be hidden in a single summary embedding.

## Provenance note
This design is **project-specific synthesis**. It is conceptually inspired by event-centric and modulation-oriented views of computation, but no single cited paper defines this store design directly.
