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
Tag: `[supporting reference]`  
Relevance: Useful for thinking about latent parameter spaces and training regimes when expert definitions are stored as compact latent artifacts.

## Executor options / few-step generation

### One-step Language Modeling via Continuous Denoising
Link: https://arxiv.org/abs/2602.16813  
Tag: `[paper-backed component]`  
Relevance: Supports the feasibility of very low-step generation strategies relevant to fast deliberation routines.

### LLaDA2.1: Speeding Up Text Diffusion via Token Editing
Link: https://arxiv.org/abs/2602.08676  
Tag: `[paper-backed component]`  
Relevance: Provides evidence for efficient diffusion-style token editing, relevant to block-diffusion executor options.

## Sparse experts / routing

### Mixture of A Million Experts (PEER)
Link: https://arxiv.org/abs/2407.04153  
Tag: `[paper-backed component]`  
Relevance: Demonstrates scaling behavior of many tiny experts, directly relevant to sparse expert assumptions in this repo.

### Sigma-MoE-Tiny Technical Report
Link: https://arxiv.org/abs/2512.16248  
Tag: `[paper-backed component]`  
Relevance: Offers practical observations about tiny MoE regimes that inform expert sizing and efficiency tradeoffs.

### DirMoE: Dirichlet-routed Mixture of Experts
Link: https://arxiv.org/abs/2602.09001  
Tag: `[paper-backed component]`  
Relevance: Informs probabilistic routing approaches and uncertainty-aware expert selection strategies.

## Async modulation / self-referential systems

### Neuro-Vesicles: Neuromodulation Should Be a Dynamical System, Not a Tensor Decoration
Link: https://arxiv.org/abs/2512.06966  
Tag: `[supporting reference]`  
Relevance: Motivates treating modulation as dynamical state, which conceptually supports asynchronous control signals in runtime design.

### Hypernetworks That Evolve Themselves
Link: https://arxiv.org/abs/2512.16406  
Tag: `[supporting reference]`  
Relevance: Provides ideas on self-referential parameter generation that are relevant to generated behavior pathways from persistent anchors.

## Workspace / online adaptation / long context

### Perceiver IO
Link: https://arxiv.org/abs/2107.14795  
Tag: `[background]`  
Relevance: Useful as a reference for flexible input/output interface design across modalities.

### Learning to (Learn at Test Time): RNNs with Expressive Hidden States
Link: https://arxiv.org/abs/2407.04620  
Tag: `[supporting reference]`  
Relevance: Supports discussions of online adaptation and expressive state transitions during inference-time updates.

### MesaNet: Sequence Modeling by Locally Optimal Test-Time Training
Link: https://arxiv.org/abs/2506.05233  
Tag: `[supporting reference]`  
Relevance: Informs local adaptation strategies that may be adapted for offline bootstrap experiments.

### Recursive Language Models
Link: https://arxiv.org/abs/2512.24601  
Tag: `[background]`  
Relevance: Provides context for recursive decomposition and multi-pass processing relevant to deliberation routines.

## Continuous depth / adaptive compute / efficient attention

### Neural ODEs
Link: https://arxiv.org/abs/1806.07366  
Tag: `[background]`  
Relevance: Foundational reference for continuous-depth thinking, useful in reasoning about adaptive step counts.

### Self-Attention at Constant Cost per Token via Symmetry-Aware Approximation
Link: https://arxiv.org/abs/2602.00294  
Tag: `[supporting reference]`  
Relevance: Relevant to efficient token processing constraints when deliberation budgets are tight.

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
Tag: `[supporting reference]`  
Relevance: Candidate optimization reference for optional bootstrap appendices on efficient update schemes.

### Muon is Scalable for LLM Training
Link: https://arxiv.org/abs/2502.16982  
Tag: `[supporting reference]`  
Relevance: Useful optional reference for optimizer discussions in offline initialization studies.
