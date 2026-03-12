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

## unified latents (UL)
A deterministic-latent diffusion codec pattern where an encoder emits a clean latent, fixed noise is injected to define a modeled latent, and paired prior/decoder diffusion models optimize bitrate versus reconstruction quality.

## flow-map language model (FMLM)
A distilled few-step executor that approximates larger integration jumps from a trained continuous flow language model, enabling one-step or low-step decoding.

## decoding-error-rate schedule
A time reparameterization that allocates training and sampling effort to trajectory regions where token identities are actively resolving.

## unmasking set (Γ_t)
In editable block diffusion, masked token positions whose top prediction confidence exceeds the unmasking threshold and are therefore filled this step.

## editing set (Δ_t)
In editable block diffusion, already-filled positions where a different token prediction exceeds the edit threshold and is therefore replaced this step.

## product-key routing
Sparse retrieval method that splits a query into subqueries, retrieves top matches from sub-key tables, and reconstructs candidate keys efficiently before final top-k selection.

## tiny expert
A minimal specialist module (for example singleton MLP form `e_i(x) = σ(u_i^T x)v_i`) designed for massive sparse composition rather than standalone depth.

## expected-k sparsity control
Routing constraint that penalizes deviation between expected active experts and a target `k`, exposing an explicit compute/sparsity knob.

## event-driven modulation
A control pattern where transient entities are emitted, migrate, dock, release local effects, and decay, instead of applying global persistent modulation.

## self-referential bootstrap
Offline initialization loop where a model (or hypernetwork) generates stochastic updates to copies of itself, evaluates descendants, and selects viable lineages.


## latent workspace
A fixed-size internal array used as a bounded read/process/write compute surface, typically filled from arbitrary inputs via cross-attention and read out through query-driven decoding.

## output query array
A structured set of vectors that encodes what output to extract (for example position, modality, task, or entity) when attending from outputs back into a latent workspace.

## test-time training (TTT) hidden state
A hidden-state design where the state is model parameters updated online during inference with self-supervised learning steps, rather than a static recurrent vector.

## adaptive solver depth
Inference-time policy that varies local optimization iterations (for example conjugate-gradient steps) per token/head/sequence based on stopping criteria such as residual error.

## out-of-core recursive reasoning
A context-management pattern where large inputs remain in an external environment and the model recursively decomposes/solves subproblems through tool-mediated interaction instead of loading all context into one window.

## continuous-depth execution (Neural ODE style)
An execution pattern where hidden state evolves under learned differential dynamics and a numerical ODE solver determines step count adaptively from error tolerances.

## SATA polynomial state
A fixed-size attention summary state built from truncated Taylor-kernel polynomial features, replacing context-length-dependent KV cache growth for constant-cost-per-token access.

## innovation number (NEAT)
A historical marker attached to structural genes so crossover can align homologous connections across genomes with different topologies.

## speciation distance (NEAT)
A compatibility metric over excess/disjoint genes and average weight differences used to cluster evolving networks and protect emerging innovations.

## goodness projection (Mono-Forward)
A local layer output produced by projecting activations into class-conditioned goodness scores and training each layer with local cross-entropy.

## Muon update
A matrix-optimizer step that orthogonalizes momentum-like matrix gradients via Newton–Schulz iteration, then applies shape-aware scaling and weight decay before updating parameters.

## term introduction checklist
A required checklist for adding new terminology in spec docs.

- **Project synthesis:** term appears first in an implementation-facing sentence before abstract discussion.
- **Project synthesis:** term is added to this glossary in the same change with a concise, simulator-relevant definition.
- **Project synthesis:** introducing spec section labels substantive claims as Paper-backed component, Project synthesis, or Open question.
- **Project synthesis:** related docs are cross-linked with relative links, including the source spec where the term is first used.
- **Project synthesis:** if term implies a literature-backed mechanism, bibliography coverage is verified or updated in the same change.

## generated routine
Transient L2/L3 behavior instance produced for a tick from L1 state and current context. Use this term in runtime behavior descriptions instead of generic "executor" unless specifically naming a modality executor family (for example flow-map or block diffusion).

## modality executor
A concrete execution family (for example flow-map language model or block diffusion editor) used to realize generated routines for a modality-specific output path.

## L1 record
The full persisted per-address state envelope validated and written at commit boundaries (schema version, routing profile, safety state, provenance, and lifecycle metadata).

## changed paths
A compact list of field paths in L1 that were modified by a candidate delta in a tick, used for replay/debug and commit auditing.
