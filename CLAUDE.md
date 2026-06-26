# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is a **skill-authoring repository**, not an application. The deliverable is a group of 9 **Agent Skills** (markdown) that teach a coding agent to build *isomorphic architecture-intent documents* and run *architecture-level review* against them. There is no build, compile, lint, or test step — the "source" is the `SKILL.md` files plus the shared conventions under `skills/arch-docs-conventions/`. "Running" a skill means an agent (Claude Code, Codex, Cursor, Copilot, Gemini, Kimi) loads it.

Validation is manual and lives in `_baseline/` (git-ignored — it contains excerpts of a private codebase; never publish it or reference its contents in distributed files). Use it as before/after fixtures when changing skill behavior; don't add it to `package.json` `files` or commit it.

## The skill group (architecture)

The nine skills form a closed loop around a **WHAT / WHY ownership split** — the central design axis. Internalize it before editing any skill:

- **WHAT** (structural facts: components, boundaries, seams) is written by `build`/`update` from *landed code only* — never ahead of code, no history/process notes.
- **WHY + strength markers** (`🧱` hard structural · `💡` recommended · `🚧` tentative) are **owned by a human**, captured *only* through `arch-why-elicit` by questioning + bidirectional code grounding. No other skill may invent or write WHY.
- **`⏳`** = WHAT written, marker/WHY pending. It occupies the *same slot* as the marker in a boundary cell. `arch-why-elicit` is the **only** skill that clears `⏳`.

Roles (see `README.md` and `skills/arch-docs-conventions/SKILL.md` for the full table):

- **Atoms** — `arch-doc-build` (generative: fill blanks, all markers `⏳`), `arch-doc-update` (conservative: minimal patch, preserve human WHY, delete what code deleted), `arch-why-elicit` (interactive WHY elicitation), `arch-spec-review` (**read-only** compliance check — reports drift as conflict artifacts + reconciliation signals; changes nothing, decides nothing), `arch-doc-reconcile` (**read-only** — identifies code↔document divergence and produces a *suggestion report* of doc changes; writes nothing, dispatches nothing, decides nothing).
- **Orchestrators** — `arch-doc-orchestrate` (parallel subagent fan-out to build/refresh the WHAT tree), `arch-why-orchestrate` (sequential, human-in-the-loop WHY interview).
- **Conductor** — `arch-spec-flow` weaves the two architecture touchpoints into a spec-driven-development loop.

**`arch-spec-review` (check) and `arch-doc-reconcile` (suggest) are deliberately split**, and **both are read-only report producers** — neither edits a document or dispatches anything. Review reports drift; reconcile decides *what the document change should be* and emits it as a suggestion. The conductor (`arch-spec-flow`) relays reconcile's suggestions to the developer, who adopts them; only then do `arch-doc-update` / `arch-doc-build` write. Don't fold the doc-change decision back into review, and don't give reconcile the power to edit or dispatch — that authority lives in the flow + the human. The conductor dispatches review and reconcile on the **best available model** (their judgment is high-stakes); the WHAT-writing atoms may run cheaply.

**`build` and `update` are deliberately separate skills** with opposite disciplines. Do not merge them or cross-pollinate their instructions: a combined skill drifts at runtime and degenerates into "regenerate", overwriting human WHY. Keep each skill's instruction set single-purpose and non-conflicting.

`arch-spec-review` is gated by a **strict completeness gate**: every document it consumes must be entirely `⏳`-free or it halts. This gate is a deliberate forcing function — do not soften it to "skip pending items".

The whole set is **script-free and agent-enforced**. The only deterministic check is the optional `enforced_by` hook, attached *only* when a real static/dynamic check exists. Don't introduce mandatory tooling.

## Shared conventions — single source of truth

`skills/arch-docs-conventions/` is the group's home and holds the two dependencies every other skill reads:

- `assets/template.md` — the fill-in skeleton every document unit uses (this is what makes the tree *isomorphic*; all units share one skeleton).
- `references/intent-contract.md` — the machine-readable writing/parsing rules (boundaries/invariants, strength markers, stable ids, `⏳`, seams/drill-down, footprint/blast-radius, completeness gate, `enforced_by`, conflict artifact, document-language default).

When changing how documents are written, parsed, or marked, **edit the contract/template here first**, then reconcile the consuming skills. Do not encode divergent rules inside individual `SKILL.md` files.

## Editing constraints (these break things silently)

- **Siblings must sit together.** Skills cross-reference the home skill with relative paths `../arch-docs-conventions/assets/template.md` and `../arch-docs-conventions/references/intent-contract.md`. Never rename `arch-docs-conventions`, never move a skill out of `skills/`, and never ship one skill alone — installing a subset breaks these references.
- **Frontmatter is YAML — watch colons in `description`.** A past bug (commit `7b325c2`) was a colon inside `arch-spec-review`'s `description` breaking YAML parsing. Keep `name`/`description` as clean single-line scalars; quote or rephrase if the text needs a colon.
- **`description` is the trigger.** Each skill fires off its frontmatter `description`. Recent commits tuned these for over/under-triggering (e.g. `8d3f905`, `cd281f3`). Edit descriptions surgically and consider trigger overlap/gaps across the group.

## Distribution

Per-runtime manifests must stay in sync when versioning or renaming: `.claude-plugin/` (plugin + marketplace), `.codex-plugin/`, `.cursor-plugin/`, `.kimi-plugin/`, plus `package.json`. The canonical repo slug referenced in docs/manifests is `Gloria-GK-406/sdd-arch-skill`. The `skills/` directory is the unit that gets installed (via `npx skills`, native plugin marketplace, or copying into the agent's skills dir).
