# Annotated Bibliography

Related docs: [architecture](architecture.md), [runtime](runtime.md), [training bootstrap](training-bootstrap.md), [research synthesis](research-synthesis.md), [open questions](open-questions.md).

This bibliography groups references by architectural role. Tags indicate how each source is used here:
- `[paper-backed component]` = direct support for a component class.
- `[supporting reference]` = informs design choices but not a direct one-to-one blueprint.
- `[background]` = useful conceptual context.

## Weight-space generation / neural manifold

### Diffusion-Based Neural Network Weights Generation
Link: https://arxiv.org/abs/2402.18153
Tag: `[paper-backed component]`
Relevance: Demonstrates conditional latent-space generation of network weights from context and supports the project pattern for context-conditioned expert spawning.

### DeepWeightFlow: Re-Basined Flow Matching for Generating Neural Network Weights
Link: https://arxiv.org/abs/2601.05052
Tag: `[paper-backed component]`
Relevance: Demonstrates direct flow matching in full weight space with symmetry-aware preprocessing, supporting fast direct generation of runnable expert weights.

### Flows and Diffusions on the Neural Manifold
Link: https://arxiv.org/abs/2507.10623
Tag: `[supporting reference]`
Relevance: Frames weight generation as manifold-aware trajectory modeling, supporting L1 evolution strategies that use training dynamics as priors.

### Unified Latents (UL): How to train your latents
Link: https://arxiv.org/abs/2602.17270
Tag: `[paper-backed component]`
Relevance: Provides a concrete encoder-prior-decoder diffusion codec recipe with explicit bitrate controls, supporting reconstructible latent codecs for heavy internal artifacts.

## Executor options / few-step generation

### One-step Language Modeling via Continuous Denoising
Link: https://arxiv.org/abs/2602.16813
Tag: `[paper-backed component]`
Relevance: Establishes one-hot continuous flow denoising with decoding-error-rate scheduling and flow-map distillation, supporting replayable one-step/few-step text executors.

### LLaDA2.1: Speeding Up Text Diffusion via Token Editing
Link: https://arxiv.org/abs/2602.08676
Tag: `[paper-backed component]`
Relevance: Provides dual-threshold draft-and-edit decoding, mixed M2T/T2T training, and speed/quality operating modes for editable block executors.

## Sparse experts / routing

### Mixture of A Million Experts (PEER)
Link: https://arxiv.org/abs/2407.04153
Tag: `[paper-backed component]`
Relevance: Demonstrates product-key retrieval with singleton experts and multi-head sparse composition, supporting the repo default for massive tiny-expert lookup with bounded active compute.

### Sigma-MoE-Tiny Technical Report
Link: https://arxiv.org/abs/2512.16248
Tag: `[paper-backed component]`
Relevance: Offers practical observations about tiny MoE regimes that inform expert sizing and efficiency tradeoffs.

### DirMoE: Dirichlet-routed Mixture of Experts
Link: https://arxiv.org/abs/2602.09001
Tag: `[paper-backed component]`
Relevance: Provides a differentiable routing recipe that disentangles expert selection and contribution via Gumbel-Sigmoid + Dirichlet with explicit expected-k sparsity control.

## Async modulation / self-referential systems

### Neuro-Vesicles: Neuromodulation Should Be a Dynamical System, Not a Tensor Decoration
Link: https://arxiv.org/abs/2512.06966
Tag: `[paper-backed component]`
Relevance: Specifies entity-based modulation dynamics (emit/migrate/dock/release/decay) and differentiable density relaxations, supporting the repo async modulation overlay.

### Hypernetworks That Evolve Themselves
Link: https://arxiv.org/abs/2512.16406
Tag: `[paper-backed component]`
Relevance: Defines self-referential stochastic hypernetwork updates with heritable mutation controls, supporting bootstrap-phase generation of viable self-updating L1 seeds.

## Workspace / online adaptation / long context

### Perceiver IO
Link: https://arxiv.org/abs/2107.14795
Tag: `[paper-backed component]`
Relevance: Establishes latent read/process/write architecture with query-driven output decoding and output-query subsampling strategies, supporting bounded multimodal ingress/egress workspaces.

### Learning to (Learn at Test Time): RNNs with Expressive Hidden States
Link: https://arxiv.org/abs/2407.04620
Tag: `[paper-backed component]`
Relevance: Defines hidden state as inference-time trainable learner parameters with learned view projections, mini-batch updates, and dual-form acceleration for online adaptation.

### MesaNet: Sequence Modeling by Locally Optimal Test-Time Training
Link: https://arxiv.org/abs/2506.05233
Tag: `[paper-backed component]`
Relevance: Provides a locally optimal regression-based recurrent layer with dynamic conjugate-gradient solve depth, supporting adaptive solver-based reasoning at test time.

### Recursive Language Models
Link: https://arxiv.org/abs/2512.24601
Tag: `[paper-backed component]`
Relevance: Demonstrates REPL-mediated recursive decomposition over externalized context with constant-size root history, supporting out-of-core reasoning workflows.

## Continuous depth / adaptive compute / efficient attention

### Neural ODEs
Link: https://arxiv.org/abs/1806.07366
Tag: `[paper-backed component]`
Relevance: Establishes continuous-depth dynamics, adjoint sensitivity training, and tolerance-controlled adaptive compute, supporting solver-governed expert execution and continuous-time latent dynamics.

### Self-Attention at Constant Cost per Token via Symmetry-Aware Approximation
Link: https://arxiv.org/abs/2602.00294
Tag: `[paper-backed component]`
Relevance: Defines symmetry-aware Taylor-feature attention with fixed-size accumulated state, supporting constant-cost long-history access without context-length KV growth.

### Gated Attention for Large Language Models
Link: https://arxiv.org/abs/2505.06708
Tag: `[supporting reference]`
Relevance: Supports selective compute allocation concepts aligned with sparse scheduling goals.

## Bootstrap / optimization / optional appendices

### Evolving Neural Networks through Augmenting Topologies (NEAT)
Link: https://nn.cs.utexas.edu/?stanley%3Aec02=
Tag: `[paper-backed component]`
Relevance: Serves as a concrete bootstrap method for discovering initial structures before live runtime constraints apply.

### Mono-Forward
Link: https://arxiv.org/abs/2501.09238
Tag: `[paper-backed component]`
Relevance: Provides a forward-only local layerwise learning rule via goodness projections, supporting modular helper experts that train from local signals without global backprop.

### Muon is Scalable for LLM Training
Link: https://arxiv.org/abs/2502.16982
Tag: `[paper-backed component]`
Relevance: Specifies orthogonalized Newton–Schulz matrix updates with shape-aware scaling and weight decay, supporting scalable optimization of matrix-heavy modules.
