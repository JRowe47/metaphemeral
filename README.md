# Metaphemeral Network

Metaphemeral Network is a research project for a self-building neural architecture where only per-address L1 state is persistent, while most higher-order behavior is generated at runtime.

## What it is
This repo defines a documentation-first architecture for a fixed-address compute substrate that supports:
- persistent local state,
- ephemeral behavior generation,
- sparse message-driven compute,
- asynchronous reflex and deliberation execution.

The intent is to make the runtime model precise enough that contributors can implement a simulator without guessing hidden assumptions.

## What is stable vs generated
**Stable substrate (project synthesis):**
- fixed hex address lattice,
- packet bus and explicit activation stack store,
- per-address persisted L1 state,
- two-plane scheduler (reflex + deliberation).

**Generated/ephemeral behavior (project synthesis):**
- L2/L3 behaviors instantiated from current L1 state + active context,
- routing and expert participation shaped by current traffic and pressure,
- short-lived execution traces and activation frames.

## Why this repo exists
- To separate plausible paper-backed components from project-original synthesis.
- To provide a coherent implementation roadmap for a runtime simulator.
- To track unresolved design questions without overstating claims.

## Current status
The project is a **research concept** in a **documentation-first bootstrap phase**. No full runtime implementation is assumed yet.

## Quick architecture summary
- Fixed address hex lattice as compute substrate.
- Per-cell persisted L1 state (only durable model memory).
- Ephemeral L2/L3 behaviors generated on demand.
- Packet bus + activation stack store for explicit intermediate state.
- Sparse scheduling driven by local pressure and triggers.
- Async execution across reflex and deliberation planes.

See details in:
- [Architecture spec](docs/specs/architecture.md)
- [Runtime spec](docs/specs/runtime.md)
- [Activation store spec](docs/specs/activation-store.md)
- [Training bootstrap](docs/specs/training-bootstrap.md)
- [Research synthesis](docs/specs/research-synthesis.md)
- [Tick trace spec](docs/specs/tick-trace.md)
- [Open questions](docs/specs/open-questions.md)
- [Decision log](docs/specs/decision-log.md)
- [Glossary](docs/specs/glossary.md)
- [Annotated bibliography](docs/specs/bibliography.md)

## Repo map
- `AGENTS.md` — stable contributor/agent guidance.
- `README.md` — project overview for new readers.
- `docs/specs/architecture.md` — system-level architecture model.
- `docs/specs/runtime.md` — cycle semantics, scheduling, routing.
- `docs/specs/activation-store.md` — explicit activation caching model.
- `docs/specs/training-bootstrap.md` — staged initialization and learning plan.
- `docs/specs/research-synthesis.md` — mapping from weight-generation literature to runtime use.
- `docs/specs/tick-trace.md` — canonical one-tick walkthrough and observability contract baseline.
- `docs/specs/open-questions.md` — unresolved decisions and validation needs.
- `docs/specs/decision-log.md` — ADR-style decision records with owners, checkpoints, and fallback paths.
- `docs/specs/glossary.md` — shared terminology.
- `docs/specs/bibliography.md` — linked paper map with relevance notes.

## Disclaimer
Several core choices (fixed hex lattice, only-L1-persists rule, activation stack store, and two-plane scheduler) are **project-original synthesis**, not direct copies of any single paper. Paper references in this repo are used as support for components and design intuition, not as proof that the full architecture is validated.
