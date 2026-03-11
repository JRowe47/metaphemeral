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

## Recommended synthesis for current project stage
- **Project synthesis:** use D2NWG-style machinery for context-conditioned spawning.
- **Project synthesis:** use DeepWeightFlow-style machinery for fast full-weight generation when direct flow is cheaper.
- **Project synthesis:** use neural-manifold conditional flow ideas for persistent L1 evolution across trajectory-informed states.

## Project synthesis vs paper-backed components

### Paper-backed components
- Weight VAE or latent codec.
- Context-conditioned latent weight generation.
- Direct flow matching in full weight space.
- Canonicalization for permutation symmetry.
- Batch-norm recalibration after generation.
- Trajectory-aware flow matching.
- Reward fine-tuning by adjoint-style matching.

### Project synthesis
- Only-L1-persists runtime model.
- Fixed hex address lattice.
- Universal expert node abstraction.
- Activation stack store with explicit cached activations.
- Pressure-based scheduler with reflex/deliberation planes.
- Implicit links induced by packet traffic.

## Contributor interpretation
- D2NWG gives a practical pattern for generating weights from context.
- DeepWeightFlow gives a practical pattern for fast full-weight generation and symmetry handling.
- Neural-manifold work gives a practical pattern for using training dynamics as priors for L1 evolution.

For this repo stage:
- **Project synthesis:** L1 persistence should be trajectory-informed.
- **Project synthesis:** L2/L3 spawning should be context-conditioned.
- **Project synthesis:** micro-experts should prefer fast direct flow matching when compute budget favors it.
- **Project synthesis:** symmetry handling and post-generation calibration are part of generation pipelines, not optional cleanup.
