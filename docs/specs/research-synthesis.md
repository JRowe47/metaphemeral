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

## Recommended synthesis for current project stage
- **Project synthesis:** use D2NWG-style machinery for context-conditioned spawning.
- **Project synthesis:** use DeepWeightFlow-style machinery for fast full-weight generation when direct flow is cheaper.
- **Project synthesis:** use neural-manifold conditional flow ideas for persistent L1 evolution across trajectory-informed states.
- **Project synthesis:** use UL as the canonical latent codec for heavy internal artifacts that must be reconstructible and generatively modelable.
- **Project synthesis:** use FLM/FMLM as the preferred few-step text executor when replayability and low-latency parallel generation are priorities.
- **Project synthesis:** use LLaDA2.1 as the default editable block executor when throughput and asynchronous self-correction are priorities.

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

### Project synthesis
- Only-L1-persists runtime model.
- Fixed hex address lattice.
- Universal expert node abstraction.
- Activation stack store with explicit cached activations.
- Pressure-based scheduler with reflex/deliberation planes.
- Implicit links induced by packet traffic.
- Runtime policy for choosing codec and executor per workload class.
- Runtime thresholds that switch between speed and quality modes for editable executors.

## Contributor interpretation
- D2NWG gives a practical pattern for generating weights from context.
- DeepWeightFlow gives a practical pattern for fast full-weight generation and symmetry handling.
- Neural-manifold work gives a practical pattern for using training dynamics as priors for L1 evolution.
- UL gives a practical pattern for stable bitrate-controlled latent codecs.
- FLM/FMLM gives a practical pattern for one-step/few-step replayable text execution.
- LLaDA2.1 gives a practical pattern for draft-fast, edit-later text execution.

For this repo stage:
- **Project synthesis:** L1 persistence should be trajectory-informed.
- **Project synthesis:** L2/L3 spawning should be context-conditioned.
- **Project synthesis:** micro-experts should prefer fast direct flow matching when compute budget favors it.
- **Project synthesis:** symmetry handling and post-generation calibration are part of generation pipelines, not optional cleanup.
- **Project synthesis:** internal codecs should default to UL-style deterministic latent pipelines with explicit bitrate knobs.
- **Project synthesis:** text executors should select FLM/FMLM or LLaDA2.1 based on replayability versus editable-throughput needs.
