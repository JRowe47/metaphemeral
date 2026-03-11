# AGENTS

## Purpose of this guide
This document defines how Codex and human contributors should execute work in this repository to maximize quality, velocity, and reproducibility.

## Operating model (project-management first)
Use this workflow for every non-trivial task:

1. **Define outcome first**
   - Write a one-sentence objective and explicit completion criteria before editing files.
   - Prefer measurable criteria (e.g., “spec section updated with assumptions, failure modes, and bibliography links”).

2. **Plan in small, verifiable steps**
   - Break work into the smallest independent units that can be reviewed.
   - Keep exactly one step in progress at a time.
   - Re-plan when findings invalidate assumptions.

3. **Execute with narrow diffs**
   - Change only files required for the objective.
   - Avoid incidental refactors unless they unblock delivery.
   - Keep commits scoped so each commit has one clear intent.

4. **Validate before handoff**
   - Run available checks relevant to changed files.
   - Manually verify links, terminology consistency, and assumptions.
   - Document any unvalidated risk explicitly.

5. **Close with traceable delivery**
   - Summarize what changed, why, and how it was validated.
   - Record follow-up items separately from completed scope.

## Repo purpose
Metaphemeral Network is a research repository for a self-building neural architecture where only per-address **L1 state** persists. Runtime behaviors (L2/L3), routing patterns, and most intermediate computation are generated on demand.

## Source-of-truth layout
- `docs/specs/` is the source of truth for architecture and runtime behavior.
- `README.md` is newcomer-facing orientation.
- `AGENTS.md` is operational guidance for contributors and coding agents.

## Content classification policy (required)
Every substantive architecture/runtime claim must be labeled as one of:
- **Paper-backed component**
- **Project synthesis**
- **Open question**

Additional requirements:
1. If you add or revise a literature-backed claim, update `docs/specs/bibliography.md` in the same change.
2. Do not present speculative extrapolation as established fact.
3. Preserve the **only-L1-persists** assumption unless the change explicitly revises core architecture.

## Documentation quality standards
When editing docs/specs:
- Lead with plain English, then provide structured technical detail.
- Prefer implementation-oriented language over conceptual prose.
- State assumptions, constraints, and failure modes explicitly.
- Keep diagrams concise (Mermaid preferred).
- Cross-link related docs using relative links.
- Update glossary entries when introducing new terms.

## Scope boundaries (current project phase)
- Repository is documentation-first and pre-implementation.
- Do not add fake APIs, placeholder code, or implied implementation status.
- Keep runtime definitions concrete enough for simulator implementation planning.

## Decision and change hygiene
- Prefer small, composable sections over large rewrites.
- Keep top-level docs readable and navigable.
- Ensure cross-links are not broken.
- Capture unresolved decisions as **Open question** entries with a validation path.

## Codex execution best practices
To get best results from Codex in this repository:

1. **State constraints explicitly**
   - Provide scope, target files, and forbidden changes.
   - Call out required deliverables (e.g., commit, PR body, checklist).

2. **Favor iterative delivery**
   - Ask for a brief plan before deep edits when scope is ambiguous.
   - Approve direction early by checking section headings and structure first.

3. **Require evidence-based completion**
   - Ask for exact commands run and outputs summarized.
   - Require file/line citations in final summaries.

4. **Enforce quality gates**
   - Require classification labels (Paper-backed / Synthesis / Open question) in spec changes.
   - Require bibliography updates for literature-linked claims.

5. **Keep PRs reviewable**
   - One objective per PR.
   - Clear title, concise rationale, explicit validation notes.

## Pull request message expectations
Each PR should include:
- Objective and completion criteria.
- Scope (files changed).
- Validation performed (commands/checks/manual verification).
- Risks or open follow-ups.
