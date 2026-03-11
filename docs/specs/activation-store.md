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
1. Drop expired frames first.
2. Evict lowest-value blocked frames.
3. Demote non-critical deliberation tasks.
4. Emit overflow signal packet.
5. If unresolved, trigger reflex quarantine mode.

**Open question:** optimal eviction objective under mixed-modal loads.

## Why this is not a simple vector summary
A vector summary collapses temporal and dependency structure into one representation. The activation stack store retains explicit, queryable, and schedulable units with lineage and control metadata. This supports fine-grained rollback, debugging, and pressure-aware scheduling that would be hidden in a single summary embedding.

## Provenance note
This design is **project-specific synthesis**. It is conceptually inspired by event-centric and modulation-oriented views of computation, but no single cited paper defines this store design directly.
