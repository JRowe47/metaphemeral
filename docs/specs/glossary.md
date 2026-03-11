# Glossary

Related docs: [README](../../README.md), [architecture](architecture.md), [runtime](runtime.md), [activation store](activation-store.md), [bibliography](bibliography.md).

## metaphemeral
A design stance where durable state is minimal and localized, while most higher-order behavior is generated transiently at runtime.

## L1 / L2 / L3
Behavioral levels in this project:
- **L1:** persisted per-address state anchor.
- **L2:** fast generated routines for local adaptation and routing support.
- **L3:** deeper generated routines for multi-step deliberation/synthesis.

## persisted state
State that survives commit boundaries and remains available in future ticks. In this repo, this is restricted to per-address L1.

## universal expert node
A lattice cell treated as a reusable execution unit that can instantiate different behavior forms from one persistent L1 anchor.

## activation stack store
Explicit local frame store containing handles, dependencies, priorities, and trace metadata for schedulable intermediate computation.

## packet bus
Messaging substrate used to move packets between addresses and external ports.

## reflex plane
Fast scheduler plane for interrupts, safety checks, and low-latency responses.

## deliberation plane
Budgeted scheduler plane for deeper, multi-step generated routines.

## capability beacon
Metadata advertisement that signals what a node can currently process, used for group/capability routing.

## trace id
Identifier linking packets, frames, and updates belonging to the same execution lineage.

## project synthesis
A repo-specific design decision or integration that is not claimed as directly established by one paper.

## paper-backed component
A component category whose core mechanism is supported by cited literature in [bibliography](bibliography.md), while still requiring project-specific integration details.

## conditional spawning
Runtime generation of expert weights or routines from L1 state plus immediate context, rather than loading fixed pre-bound modules.

## canonicalization
Preprocessing of weight sets to reduce permutation-symmetry mismatch before fitting a generator in full weight space.

## trajectory prior
A training prior derived from optimization paths (not only final checkpoints), used to constrain or guide generated weight evolution.
