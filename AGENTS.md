# AGENTS

## Repo purpose
Metaphemeral Network is a research repository for a self-building neural architecture where only per-address **L1 state** persists. Runtime behaviors (L2/L3), routing patterns, and most intermediate computation are generated on demand.

## What belongs where
- `docs/specs/` is the source of truth for architecture and runtime behavior.
- `README.md` is newcomer-facing orientation.
- `AGENTS.md` is stable operational guidance for contributors and coding agents.

## Documentation rules (required)
1. Separate claims into one of:
   - **Paper-backed component**
   - **Project synthesis**
   - **Open question**
2. If you add or change a claim tied to literature, update `docs/specs/bibliography.md` in the same change.
3. Do not present speculative extrapolation as established fact.
4. Prefer implementation-oriented language over conceptual prose.
5. Preserve the **only-L1-persists** assumption unless a change explicitly revises it.

## Writing conventions
- Plain English first; structured technical detail second.
- Keep diagrams concise (Mermaid preferred in specs).
- No marketing language.
- Use explicit assumptions and failure modes.
- Cross-link related docs with relative links.

## Architecture labeling policy
When editing specs, add explicit markers:
- **Paper-backed component:** include linkable basis in `bibliography.md`.
- **Project synthesis:** identify as repo-specific design.
- **Open question:** state uncertainty and expected validation path.

## Scope boundaries for this repo stage
- This repo is documentation-first and pre-implementation.
- Avoid fake API docs or placeholder code claims.
- Keep runtime definitions concrete enough for simulator implementation.

## Change hygiene
- Keep top-level docs readable.
- Prefer small, composable sections.
- Update glossary terms when introducing new vocabulary.
- Ensure cross-links are not broken.
