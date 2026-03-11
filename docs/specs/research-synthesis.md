# Research Synthesis for Weight Generation and L1 Evolution

Related specs: [architecture](architecture.md), [runtime](runtime.md), [training bootstrap](training-bootstrap.md), [open questions](open-questions.md), [glossary](glossary.md), [bibliography](bibliography.md).

## Scope
This document maps external research results to Metaphemeral runtime needs. It separates **paper-backed components** from **project synthesis** and keeps the only-L1-persists rule explicit.

## Research-backed components we are borrowing

### 1) Conditional weight generation in latent space (D2NWG)
- **Paper-backed component:** D2NWG treats network weights as a generative target by vectorizing/chunking weights, learning a weight codec (VAE), encoding task or dataset context, aligning context and weight embeddings, and sampling conditional weight latents with diffusion before decoding to runnable parameters.
- **Project synthesis:** in Metaphemeral this is used for **conditional spawning** from persistent L1 state into ephemeral L2/L3 behavior.

Reusable pattern:
1. Collect a zoo of valid expert weights.
2. Learn a weight codec.
3. Learn a context encoder.
4. Sample conditional weight latents.
5. Decode to instantiated expert weights.

Keep:
- VAE over vectorized or chunked weights.
- Context/task encoder aligned to weight latents.
- Latent diffusion for conditional weight proposals.
- Chunk- or layer-wise generation for larger models.
- Optional sequential refinement over selected layers.

Do not over-import:
- **Open question:** how much of the staged D2NWG training pipeline is required in a live asynchronous runtime.
- **Project synthesis:** fixed lattice, activation stack store, and only-L1-persists are not established by D2NWG.

### 2) Direct flow matching in full weight space (DeepWeightFlow)
- **Paper-backed component:** DeepWeightFlow demonstrates direct flow matching in full weight space, with source-noise interpolation, velocity regression, ODE integration (for example RK4), canonicalization to handle permutation symmetries, and post-generation batch-norm recalibration.
- **Project synthesis:** in Metaphemeral this informs fast generation of runnable micro-expert weights when direct generation is cheaper than latent decode pipelines.

Keep:
- Direct flow matching as a fast full-weight generator.
- Canonicalization before training.
- Explicit ODE solving for generation.
- Batch-norm recalibration when needed.
- PCA-based compression only as scaling support.

Do not over-import:
- **Open question:** how to combine this with context-conditioned generation for role-specific spawning.
- **Project synthesis:** DeepWeightFlow does not define live asynchronous lattice runtime semantics.

### 3) Training-trajectory priors and neural-manifold modeling
- **Paper-backed component:** neural-manifold flow/diffusion work motivates trajectory-aware modeling over weight space, including conditional flow variants, multi-marginal objectives, optional latent coding, and reward-style adaptation of pretrained generators.
- **Project synthesis:** in Metaphemeral this supports viewing L1 as a point in a learned space of viable systems that can evolve via explicit commit paths.

Keep:
- Trajectory datasets, not only endpoint checkpoints.
- Conditional flow matching as a simple default.
- Multi-marginal variants when intermediate states matter.
- Gradient-flow style matching only when assumptions hold.
- Adjoint-style reward fine-tuning for post-hoc adaptation.

Do not over-import:
- **Open question:** whether a manifold coordinate can capture all memory-relevant information in this runtime.
- **Project synthesis:** only-L1-persists and explicit activation stack storage remain repo decisions.

### 4) Unified latent codec training (Unified Latents, UL)
- **Paper-backed component:** UL trains a deterministic encoder, a diffusion prior over latent trajectories, and a diffusion decoder jointly. The encoder emits deterministic `z_clean`, applies fixed Gaussian noise to produce `z0`, and the prior uses a matching minimum-noise floor so latent regularization becomes weighted denoising with a tight bitrate upper bound.
- **Paper-backed component:** the decoder uses a reweighted ELBO objective, typically with sigmoid weighting plus a decoder loss factor to tune reconstruction versus generative modeling quality.
- **Project synthesis:** in Metaphemeral this is the default codec recipe for compressing heavy internal objects (for example expert weights and state bundles) into reconstructible latents that can be generated on demand.

Keep:
- Deterministic encoder to `z_clean`.
- Fixed encoder-noise construction of `z0`.
- Diffusion prior trained directly on latent noise levels without additional prior-side weighting tricks.
- Diffusion decoder with reweighted ELBO.
- Bitrate control through latent shape/channel count, noise floor, sigmoid bias, and decoder loss factor.
- Optional stage-2 retraining of a downstream base model on frozen stage-1 latents.

Do not over-import:
- **Open question:** the best default bitrate operating point for each internal object type (weights, state bundles, routing summaries) remains workload-dependent and must be benchmarked.
- **Project synthesis:** the choice to keep only per-address L1 persistent state is independent from UL and must remain explicit.

### 5) Continuous flow language execution (FLM / FMLM)
- **Paper-backed component:** FLM represents language as fixed one-hot tensors in `R^(L × |V|)`, uses linear stochastic interpolation `I_t = (1-t)x0 + tx1`, and trains a denoiser `D_t(x) = E[x1 | I_t = x]` instead of regressing velocity directly.
- **Paper-backed component:** denoiser training uses tokenwise softmax plus cross-entropy on simplex-valued token distributions, with time reparameterization by decoding error rate.
- **Paper-backed component:** FMLM distills flow execution into one-step or few-step flow-map inference by first learning jump correction and then distilling into one model.
- **Project synthesis:** in Metaphemeral this is the default few-step text executor when low latency, replayability from fixed seeds, and parallel token updates are more important than autoregressive decoding.

Keep:
- Fixed one-hot continuous representation and argmax decode.
- Linear interpolant and denoiser-based objective.
- Tokenwise softmax + cross-entropy training.
- Decoding-error-rate time schedule.
- Two-stage flow-map distillation pattern for one-step/few-step execution.

Do not over-import:
- **Open question:** how robust one-step flow-map distillation remains under heavy domain shift or strict safety constraints in long-horizon tasks.
- **Project synthesis:** executor scheduling across reflex/deliberation planes remains a runtime-level policy, not a direct FLM result.

### 6) Editable block diffusion execution (LLaDA2.1)
- **Paper-backed component:** LLaDA2.1 introduces dual-set decoding with an unmasking set `Γ_t` (masked positions above `τ_mask`) and editing set `Δ_t` (already-filled positions where replacement confidence exceeds `τ_edit`), updating `Γ_t ∪ Δ_t` each step.
- **Paper-backed component:** training combines Mask-to-Token (M2T) drafting and Token-to-Token (T2T) correction, with optional Multi-turn Forward (MTF) augmentation and ELBO-based Block-level Policy Optimization (EBPO) post-training.
- **Paper-backed component:** inference supports speed/quality operating modes via threshold choices and optional multiple block editing.
- **Project synthesis:** in Metaphemeral this is the default executor where outputs should appear quickly and be refined asynchronously while other computation continues.

Keep:
- Dual-threshold draft-and-edit transition rule.
- Separate speed-focused and quality-focused decoding modes.
- Mixed M2T/T2T training objective.
- Optional MTF augmentation and EBPO post-training.
- Multiple block editing for revisiting earlier outputs.

Do not over-import:
- **Open question:** whether block-level editing introduces unacceptable instability for safety-critical outputs without additional verifier loops.
- **Project synthesis:** choosing when to stop editing versus commit is controlled by runtime policy and budget, not by the paper alone.

### 7) Extreme sparse retrieval over tiny experts (PEER)
- **Paper-backed component:** PEER introduces product-key retrieval over a massive pool of singleton MLP experts, where two-stage sub-key lookup reduces routing from naive `O(Nd)` to `O((√N + k²)d)` while preserving top-k quality.
- **Paper-backed component:** PEER uses tiny experts of the form `e_i(x) = σ(u_i^T x)v_i` and multi-head retrieval so each token composes many cheap specialists while active compute remains bounded.
- **Project synthesis:** in Metaphemeral this is the default high-cardinality addressing primitive for capability groups and lattice-wide specialist pools.

Keep:
- Product-key retrieval with sub-query decomposition.
- Tiny singleton experts as the minimum modular unit.
- Multi-head retrieval to assemble dynamic MLP behavior.
- Query normalization (or equivalent balancing control) to avoid expert collapse.
- Active-expert count as an explicit granularity/compute knob.

Do not over-import:
- **Open question:** whether singleton experts should be the only expert type or coexist with larger experts for specific runtime lanes.
- **Project synthesis:** fixed-address lattice semantics and packet routing policies are repo design choices layered on top of PEER retrieval.

### 8) Differentiable sparse probabilistic routing (DirMoE)
- **Paper-backed component:** DirMoE separates routing into differentiable expert selection and contribution, using relaxed Bernoulli (Gumbel-Sigmoid) gates for activation and a Dirichlet posterior for mixture weights.
- **Paper-backed component:** final routing is the normalized Hadamard product `normalize(z̃ ⊙ θ)` and is trained with task loss plus Dirichlet regularization and expected-k sparsity control.
- **Project synthesis:** in Metaphemeral this is the default live sparse orchestration mechanism when routing must remain end-to-end differentiable and interpretable.

Keep:
- Temperature-annealed Gumbel-Sigmoid selection gates.
- Dirichlet concentration schedules that move from exploratory to decisive routing.
- Explicit expected-k sparsity penalty to control reasoning budget.
- Concentration diagnostics for routing decisiveness.

Do not over-import:
- **Open question:** how to map expected-k controls to heterogeneous workload classes (reflex versus deliberation traffic) without destabilizing quality.
- **Project synthesis:** cluster boundaries and scheduler coupling are Metaphemeral runtime decisions, not prescribed by DirMoE.

### 9) Event-driven modulation as a dynamical overlay (Neuro-Vesicles)
- **Paper-backed component:** Neuro-Vesicles models modulation as explicit entities moving over a graph `G=(V,E)` with emission, migration, docking, release, and decay dynamics.
- **Paper-backed component:** release operators can act on activations, parameters, learning rules, or external memory, with differentiable density relaxations and RL control variants.
- **Project synthesis:** in Metaphemeral this becomes the async event layer for transient packets that locally alter behavior without violating only-L1 persistence.

Keep:
- Graph-native modulation dynamics instead of tensor-only control channels.
- Event lifecycle primitives: emit, migrate, dock, release, decay.
- Local intervention semantics rather than global overrides.
- Optional density approximation for differentiable training.

Do not over-import:
- **Open question:** which release operator classes should be enabled by default under safety constraints.
- **Project synthesis:** mailbox layout, packet schemas, and lattice hop policies remain runtime-specific design choices.

### 10) Self-referential stochastic bootstrap (Hypernetworks That Evolve Themselves)
- **Paper-backed component:** Self-Referential GHNs internalize variation generation by using a stochastic hypernetwork to produce self-updates and a deterministic branch to generate evaluation networks.
- **Paper-backed component:** node-specific mutation-rate heads and fixed random output bases provide practical mechanisms for stable self-referential evolution.
- **Project synthesis:** in Metaphemeral this is the default bootstrap strategy for discovering viable self-updating L1 seeds before live runtime rollout.

Keep:
- Graph-hypernetwork structure encoding with node embeddings + GNN processing.
- Stochastic self-update branch for offspring variation.
- Deterministic evaluation-network generation for fitness measurement.
- Heritable node-specific mutation-rate controls.

Do not over-import:
- **Open question:** what minimum evaluation suite is sufficient to certify a lineage as runtime-safe.
- **Project synthesis:** selection policy details and rollout gates into production runtime are project governance decisions.


### 11) Latent read/process/write workspaces for arbitrary I/O (Perceiver IO)
- **Paper-backed component:** Perceiver IO maps arbitrary input arrays into a fixed latent array, runs deep latent-only computation, and decodes arbitrary outputs through query-driven cross-attention.
- **Paper-backed component:** output queries encode output semantics (for example position, modality, task, or entity), allowing one shared latent workspace to support heterogeneous output formats.
- **Project synthesis:** in Metaphemeral this becomes the default ingress/egress workspace pattern for environment ports and multimodal cell boundaries where raw interface size can be very large.

Keep:
- Fixed latent workspace `z ∈ R^(N×D)` decoupled from raw input/output size.
- Input cross-attention from arbitrary input arrays into latents.
- Deep latent processing as the main compute path.
- Query-driven output decoding from latents into structured outputs.
- Output-query subsampling during training and batched full decoding at test time for very large output spaces.

Do not over-import:
- **Open question:** which default latent size and depth should be standardized per workload class (ports, sensorimotor loops, document tools).
- **Project synthesis:** the only-L1-persists rule still applies; latent workspace activity is ephemeral unless explicitly committed into L1 state.

### 12) Test-time training hidden state as an online learner (TTT layers)
- **Paper-backed component:** TTT reframes sequence hidden state as model parameters `W_t` updated during inference by self-supervised learning steps, rather than as a fixed recurrent vector.
- **Paper-backed component:** learned training/label/test views (`θ_K, θ_V, θ_Q`), mini-batch updates, dual-form acceleration, and learnable initialization/learning-rate gates provide practical mechanisms for expressive online adaptation.
- **Project synthesis:** in Metaphemeral this is the reference pattern for L1 cell state that should adapt through use, while preserving only-L1 persistence at commit boundaries.

Keep:
- Hidden state represented as learner weights `W_t`.
- Inference-time inner-loop gradient updates on local self-supervised objectives.
- Learnable view projections for training/label/test roles.
- Mini-batch and dual-form implementations for hardware-efficient updates.
- Learnable initialization and token-conditional learning-rate gates.

Do not over-import:
- **Open question:** how aggressively to allow online adaptation before stability controls must damp updates in live asynchronous settings.
- **Project synthesis:** retention, commit cadence, and rollback policy for adapted L1 state remain runtime governance decisions.

### 13) Locally optimal solver-based sequence updates (MesaNet)
- **Paper-backed component:** MesaNet defines sequence updates by solving a regularized local least-squares objective over context, with recurrent sufficient statistics and solver-based readout instead of one-step fast-weight heuristics.
- **Paper-backed component:** dynamic conjugate-gradient (CG) solve depth, chunkwise parallelization, and gating-based forgetting/write strengths provide adaptive inference-time compute with stable recurrent execution.
- **Project synthesis:** in Metaphemeral this is the primary paper-backed template for variable-depth reasoning inside experts where harder tokens/subproblems should consume more solver steps.

Keep:
- Recurrent sufficient statistics (`G_t`, `H_t`) for local optimization state.
- Approximate linear-system solves via CG for readout.
- Dynamic stopping criteria based on residual thresholds.
- Context-dependent forgetting/write gates (`γ_t`, `β_t`).
- Stability heuristics: key/query normalization, bounded regularization, and repeated-token safeguards.

Do not over-import:
- **Open question:** what default solver-stop policy best balances latency and quality under mixed reflex/deliberation budgets.
- **Project synthesis:** scheduler policy that allocates solver steps per head/token remains a runtime-level control decision.

### 14) Recursive out-of-core context interaction (Recursive Language Models)
- **Paper-backed component:** Recursive Language Models treat large prompts as external environment state and use a REPL loop where the model writes programs to inspect, transform, and recursively process context slices.
- **Paper-backed component:** constant-size root history is maintained by feeding back only metadata and tool stdout summaries, while intermediate state lives in the environment.
- **Project synthesis:** in Metaphemeral this is the default out-of-core reasoning template for long documents/logs/trajectories that exceed local model context capacity.

Keep:
- Context offloading into environment variables or external stores.
- REPL-mediated symbolic operations over external context.
- Recursive self-calls on programmatically generated subproblems.
- Constant-size root context with environment-resident intermediates.
- Programmatic decomposition as first-class behavior, not only retrieval or lossy summarization.

Do not over-import:
- **Open question:** which guardrails should gate recursive depth, tool permissions, and failure recovery in live deployments.
- **Project synthesis:** environment APIs and safety sandbox design are repo runtime concerns, not fixed by RLM alone.

## Condensed algorithm set for Metaphemeral

### Algorithm A — Conditional expert spawning (from D2NWG)
1. Build a model zoo of valid expert weights.
2. Vectorize or chunk expert weights.
3. Train a weight VAE to encode/decode weights.
4. Train a context encoder into the latent regime.
5. Align context embeddings and weight embeddings.
6. Train context-conditioned latent diffusion.
7. At runtime, sample latent weights from context, decode, and instantiate.

**Purpose in Metaphemeral:** context-conditioned creation of L2/L3 behavior from persistent L1 state.

### Algorithm B — Direct full-weight generation (from DeepWeightFlow)
1. Assemble independently trained networks.
2. Canonicalize weights to reduce permutation symmetry.
3. Flatten to one vector (or a compressed projection).
4. Sample Gaussian source `x0` and target `x1`.
5. Interpolate `xt = (1-t)x0 + tx1 + ε`.
6. Regress `vθ(xt,t)` toward velocity target `ut = x1 - x0`.
7. Integrate learned flow from noise to weight space.
8. Recalibrate batch-norm stats when architecture requires.

**Purpose in Metaphemeral:** fast generation of complete runnable micro-expert weights with explicit symmetry handling.

### Algorithm C — Trajectory-aware manifold generation (from neural-manifold work)
1. Save weight trajectories during training.
2. Optionally encode into latent space.
3. Train conditional flow models on endpoint or trajectory samples.
4. Use multi-marginal objectives if intermediate marginals are required.
5. Use gradient-flow style objectives only when justified.
6. Condition retrieval/generation on context across mixed tasks.
7. Optionally apply reward-style adjoint matching for downstream shifts.

**Purpose in Metaphemeral:** trajectory-aware and context-aware L1 evolution without full retraining.

### Algorithm D — Unified latent codec training (from UL)
1. Encode object `x` into deterministic `z_clean`.
2. Add fixed Gaussian noise to produce latent `z0`.
3. Train a diffusion prior over noisy latent trajectories with denoising-style ELBO objective.
4. Train a diffusion decoder conditioned on `z0` with reweighted ELBO.
5. Tune bitrate via latent shape, encoder noise floor, sigmoid weighting bias, and decoder loss factor.
6. Optionally freeze codec latents and retrain a downstream base generator on those frozen latents.

**Purpose in Metaphemeral:** stable codec layer for reconstructible, bitrate-controlled latent representations of heavy internal objects.

### Algorithm E — Flow-based few-step language execution (from FLM/FMLM)
1. Represent token sequence as one-hot tensor `x1`.
2. Sample Gaussian `x0` and interpolate `I_t = (1-t)x0 + tx1`.
3. Train denoiser `D_t` with tokenwise softmax + cross-entropy to predict clean language from `I_t`.
4. Recover velocity from denoiser outputs for flow integration.
5. Reparameterize time by decoding error rate.
6. Integrate the flow to sample text.
7. Distill to a flow-map model for one-step or few-step inference.

**Purpose in Metaphemeral:** low-latency text execution with replayable trajectories from fixed seeds.

### Algorithm F — Editable block-diffusion execution (from LLaDA2.1)
1. Compute token probabilities per position at each decoding step.
2. Build unmasking set `Γ_t` from masked positions above `τ_mask`.
3. Build editing set `Δ_t` from non-masked positions where replacement argmax differs and exceeds `τ_edit`.
4. Update tokens in `Γ_t ∪ Δ_t`.
5. Train with mixed M2T and T2T supervision.
6. Optionally apply MTF augmentation.
7. Optionally run EBPO post-training.
8. At inference, select Speedy or Quality thresholds and optionally enable multiple block editing.

**Purpose in Metaphemeral:** fast draft-and-repair execution where outputs can improve while parallel computation continues.


### Algorithm G — Massive sparse expert retrieval (from PEER)
1. Compute query vector `q(x)` from current token/state.
2. Split `q` into two subqueries and score against two learned sub-key tables.
3. Take top-k matches in each sub-key table.
4. Form Cartesian-product candidate keys and rerank candidates.
5. Retrieve corresponding tiny experts.
6. Compute router scores from query-key similarities.
7. Evaluate retrieved experts and aggregate weighted outputs.

**Purpose in Metaphemeral:** default high-cardinality expert retrieval primitive for large specialist pools.

### Algorithm H — Differentiable sparse routing (from DirMoE)
1. Compute router logits from token/state embedding.
2. Sample relaxed expert-selection gates with Gumbel-Sigmoid.
3. Build posterior Dirichlet parameters from gate values.
4. Sample expert-contribution weights with implicit reparameterization.
5. Form final routing weights by normalizing `z̃ ⊙ θ`.
6. Aggregate expert outputs.
7. Optimize task loss + Dirichlet regularization + expected-k sparsity loss.
8. Anneal gate temperature and concentration schedules from exploratory to decisive routing.

**Purpose in Metaphemeral:** default differentiable live-routing primitive with explicit sparsity controls.

### Algorithm I — Event-driven modulation overlay (from Neuro-Vesicles)
1. Represent runtime topology as graph `G=(V,E)`.
2. Emit modulatory entities from local activity/gradient/meta signals.
3. Move entities through `G` via learned transition kernels.
4. Sample docking decisions at candidate destinations.
5. On docking, apply local release operators (activation/parameter/rule/memory).
6. Age and decay entities so modulatory traces remain bounded.
7. Optionally train a density relaxation or RL controller over vesicle dynamics.

**Purpose in Metaphemeral:** default asynchronous signaling and local modulation primitive.

### Algorithm J — Self-referential bootstrap (from Self-Referential GHNs)
1. Encode the self-updating system as a graph hypernetwork.
2. Copy a parent instance.
3. Generate stochastic self-updates for the copy.
4. Apply updates to create offspring.
5. Generate deterministic evaluation networks for external tasks.
6. Evaluate offspring fitness and stability.
7. Select and repeat across generations, preserving heritable mutation-rate traits.

**Purpose in Metaphemeral:** default pre-live bootstrap primitive for viable self-updating seeds.


### Algorithm K — Latent workspace ingress/egress (from Perceiver IO)
1. Represent input as an arbitrary array of feature vectors.
2. Cross-attend from a fixed latent array to the input array to obtain latent state.
3. Process the latent array with repeated latent attention/MLP blocks.
4. Construct output query vectors that encode output semantics (position, modality, task, entity).
5. Cross-attend from output queries to processed latents.
6. Project resulting output array into the external target form.
7. For huge output spaces, subsample queries during training and decode in batches at test time.

**Purpose in Metaphemeral:** bounded shared workspace for high-volume multimodal ingress/egress without scaling core compute with interface size.

### Algorithm L — Persistent adaptive cell state (from TTT layers)
1. Treat hidden state as learner parameters `W_t` for local model `f`.
2. Build learned training/label/test views for current token or chunk.
3. Compute self-supervised loss on the current token/chunk.
4. Update `W_t` using one gradient step (or mini-batch update).
5. Produce current output/state from updated learner.
6. Carry updated learner parameters as next hidden state.
7. Use dual-form implementation where available for matrix-efficient inner-loop compute.

**Purpose in Metaphemeral:** L1 state that improves online from local experience instead of acting as a passive recurrent vector.

### Algorithm M — Adaptive solver-based reasoning (from MesaNet)
1. Compute keys, queries, values, and gating strengths from current input/state.
2. Update recurrent sufficient statistics `G_t` and `H_t`.
3. Solve `(H_t + Λ) q*_t = q_t` approximately with conjugate gradient.
4. Compute output `o_t = G_t q*_t`.
5. Stop solver early when residual falls below threshold.
6. Use chunkwise parallelization for training/inference throughput.
7. Allow solver iterations to vary per head, sequence, and token.

**Purpose in Metaphemeral:** adaptive reasoning depth where experts spend additional test-time compute only when local difficulty warrants it.

### Algorithm N — Out-of-core recursive context handling (from Recursive Language Models)
1. Place full external context into an environment variable/store.
2. Give root model compact metadata plus REPL/tool interface.
3. Let model write code to inspect, chunk, filter, and transform context.
4. Allow recursive model calls on derived subproblems/chunks.
5. Store intermediates in environment rather than root context window.
6. Continue loop until final output variable is set.
7. Return final variable as answer.

**Purpose in Metaphemeral:** scalable handling of contexts larger than local windows via recursive environment-coupled computation.

## Recommended synthesis for current project stage
- **Project synthesis:** use D2NWG-style machinery for context-conditioned spawning.
- **Project synthesis:** use DeepWeightFlow-style machinery for fast full-weight generation when direct flow is cheaper.
- **Project synthesis:** use neural-manifold conditional flow ideas for persistent L1 evolution across trajectory-informed states.
- **Project synthesis:** use UL as the canonical latent codec for heavy internal artifacts that must be reconstructible and generatively modelable.
- **Project synthesis:** use FLM/FMLM as the preferred few-step text executor when replayability and low-latency parallel generation are priorities.
- **Project synthesis:** use LLaDA2.1 as the default editable block executor when throughput and asynchronous self-correction are priorities.
- **Project synthesis:** use PEER-style product-key retrieval for massive tiny-expert pools and high-cardinality capability addressing.
- **Project synthesis:** use DirMoE-style routing for calibrated, differentiable sparse selection with explicit expected-k control.
- **Project synthesis:** use Neuro-Vesicles-style event entities for asynchronous local modulation above the base network graph.
- **Project synthesis:** use Self-Referential GHN-style internalized evolution for offline bootstrap of viable L1 lineages.
- **Project synthesis:** use Perceiver IO as the default latent ingress/egress workspace at environment and multimodal boundaries.
- **Project synthesis:** use TTT-style hidden-state learning when persisted L1 state should adapt online through self-supervised updates.
- **Project synthesis:** use Mesa-style local optimization with dynamic solver stops for variable-depth reasoning inside active experts.
- **Project synthesis:** use RLM-style recursive environment interaction for out-of-core contexts that exceed local windows.

## Project synthesis vs paper-backed components

### Paper-backed components
- Weight VAE or latent codec.
- Context-conditioned latent weight generation.
- Direct flow matching in full weight space.
- Canonicalization for permutation symmetry.
- Batch-norm recalibration after generation.
- Trajectory-aware flow matching.
- Reward fine-tuning by adjoint-style matching.
- Deterministic-latent diffusion codec training with bitrate controls (UL).
- Continuous one-hot flow denoising and flow-map distillation for few-step generation (FLM/FMLM).
- Dual-threshold block draft-and-edit diffusion execution (LLaDA2.1).
- Product-key routing over massive tiny-expert pools (PEER).
- Differentiable selection/contribution routing with expected-k sparsity controls (DirMoE).
- Event-entity modulation dynamics over graph topology (Neuro-Vesicles).
- Self-referential stochastic hypernetwork updates for evolutionary bootstrap (Hypernetworks That Evolve Themselves).
- Perceiver IO latent read/process/write workspace with query-driven output decoding.
- TTT-style hidden state as an inference-time trainable learner.
- MesaNet-style locally optimal solver-based sequence update with dynamic CG depth.
- Recursive Language Model style REPL-mediated out-of-core recursion over environment context.

### Project synthesis
- Only-L1-persists runtime model.
- Fixed hex address lattice.
- Universal expert node abstraction.
- Activation stack store with explicit cached activations.
- Pressure-based scheduler with reflex/deliberation planes.
- Implicit links induced by packet traffic.
- Runtime policy for choosing codec and executor per workload class.
- Runtime thresholds that switch between speed and quality modes for editable executors.
- Mapping sparse-routing controls onto reflex versus deliberation budget policy.
- Runtime packet schemas and safety constraints for event-driven modulation entities.
- Certification thresholds for promoting bootstrap lineages into live deployment.
- Mapping ingress/egress interfaces to shared latent workspace schemas across ports.
- Policy limits for online L1 adaptation rates and commit frequency.
- Runtime residual thresholds and iteration caps for solver-based reasoning loops.
- Tool permissions, recursion-depth caps, and watchdog behavior for out-of-core recursive loops.

## Contributor interpretation
- D2NWG gives a practical pattern for generating weights from context.
- DeepWeightFlow gives a practical pattern for fast full-weight generation and symmetry handling.
- Neural-manifold work gives a practical pattern for using training dynamics as priors for L1 evolution.
- UL gives a practical pattern for stable bitrate-controlled latent codecs.
- FLM/FMLM gives a practical pattern for one-step/few-step replayable text execution.
- LLaDA2.1 gives a practical pattern for draft-fast, edit-later text execution.
- PEER gives a practical pattern for efficient lookup across huge pools of tiny experts.
- DirMoE gives a practical pattern for differentiable, interpretable sparse routing with explicit control knobs.
- Neuro-Vesicles gives a practical pattern for asynchronous local modulation via event entities.
- Self-Referential GHNs give a practical pattern for internalized variation during bootstrap evolution.
- Perceiver IO gives a practical pattern for bounded latent workspaces that can read/write arbitrary arrays.
- TTT layers give a practical pattern for hidden states that learn online during inference.
- MesaNet gives a practical pattern for local optimal solves with adaptive per-token compute.
- Recursive Language Models give a practical pattern for recursive out-of-core context handling through environment tools.

For this repo stage:
- **Project synthesis:** L1 persistence should be trajectory-informed.
- **Project synthesis:** L2/L3 spawning should be context-conditioned.
- **Project synthesis:** micro-experts should prefer fast direct flow matching when compute budget favors it.
- **Project synthesis:** symmetry handling and post-generation calibration are part of generation pipelines, not optional cleanup.
- **Project synthesis:** internal codecs should default to UL-style deterministic latent pipelines with explicit bitrate knobs.
- **Project synthesis:** text executors should select FLM/FMLM or LLaDA2.1 based on replayability versus editable-throughput needs.
- **Project synthesis:** sparse society phases should pair PEER retrieval with DirMoE probabilistic routing where gradients must flow through selection.
- **Project synthesis:** asynchronous control should use event entities with bounded lifetime rather than persistent global modifiers.
- **Project synthesis:** bootstrap phases should prioritize self-referential variation loops before enabling live runtime mutation.
- **Project synthesis:** system edges should default to Perceiver IO latent workspaces for large or multimodal ingress/egress.
- **Project synthesis:** cells should use TTT-style adaptation when persisted L1 must improve from ongoing local experience.
- **Project synthesis:** experts should use Mesa-style solver loops when variable reasoning depth is worth extra FLOPs.
- **Project synthesis:** out-of-core problems should use RLM-style recursive environment interaction instead of forcing full context into root windows.
