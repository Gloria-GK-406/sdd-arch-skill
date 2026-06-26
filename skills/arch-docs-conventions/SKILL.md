---
name: arch-docs-conventions
description: Use when starting to build, update, fill rationale for, or review architecture-intent documents with this skill group, or when another arch-* skill needs the shared template or the machine-readable conventions (intent-contract) that the group depends on.
---

# arch-docs-conventions · Architecture documentation skill group

## Overview

This skill group maintains a tree of **isomorphic** architecture-intent documents so an AI can recognize and stop architectural drift early. The goal is not to keep the architecture perfectly clean, but to preserve cleanliness at a chosen scale so the cost of later targeted refactoring stays affordable (it keeps the "refactoring window" from closing).

This skill is the group's **entry point and home of the shared conventions**. It does not produce or review documents itself; it points to the other skills and hosts the two shared dependencies they all rely on.

## Shared dependencies (hosted here)

- **`assets/template.md`** — the fill-in template for an architecture document. Every unit (overview or component) uses the same skeleton, so the tree is isomorphic.
- **`references/intent-contract.md`** — the machine-readable conventions: the writing and parsing rules for boundaries, strength markers, ids, `⏳`, seams, code pointers, the completeness gate, the conflict artifact, and the document-language default. `arch-doc-build`/`arch-doc-update` write per this contract; `arch-why-elicit` clears `⏳` per it; `arch-spec-review`/`arch-doc-orchestrate` parse per it.

Other skills reference these as `../arch-docs-conventions/assets/template.md` and `../arch-docs-conventions/references/intent-contract.md`.

## The skills

**Atoms (single discipline):**
| skill | concern | discipline |
|---|---|---|
| `arch-doc-build` | whether it exists | if absent, fill WHAT from scratch; leave every marker `⏳` |
| `arch-doc-update` | whether it changed | if changed, minimal patch; preserve human WHY; new points `⏳`; delete what the code deleted |
| `arch-why-elicit` | why | elicit WHY by questioning + bidirectional code grounding; the only skill that clears `⏳`; asks/records/verifies, never invents |
| `arch-spec-review` | whether it conforms | architecture-level subtraction on a landed spec; makes no architectural decisions; gated on a `⏳`-free document |

**Orchestrators (conduct, do no work themselves):**
| skill | function | parallelism |
|---|---|---|
| `arch-doc-orchestrate` | traverse the tree, route each unit (build/update/skip), dispatch subagents, stitch | parallel fan-out (different files) |
| `arch-why-orchestrate` | scan the tree for `⏳`, order foundation-first, drive `arch-why-elicit`, cross-check rationale consistency | sequential, human-in-the-loop (no fan-out) |

## Ownership split

- **WHAT** (structural facts): written by build/update.
- **WHY + `🧱/💡/🚧` markers**: written and owned by a human, captured through `arch-why-elicit` by questioning and grounded against code.
- **`⏳`**: WHAT written, marker and WHY pending; in the boundary cell `⏳` occupies the same slot as the marker.
- Enforcement is **agent-based, script-free**; the only opt-in deterministic check is `enforced_by` (attached only when a real static/dynamic check exists).

## Closed loop

```
Project level: arch-doc-orchestrate (parallel build/update → whole WHAT tree, all ⏳)
        ▼
   arch-why-orchestrate (sequential human interview via arch-why-elicit; fill WHY; cross-document consistency)
        ▼
   Document tree ⏳-free (WHAT + WHY complete)
        │
Spec landed ──────────▶ arch-spec-review (completeness gate: must be ⏳-free, else halt) → architecture-level subtraction
                            │ if an architectural change is introduced
                            ▼
                         arch-doc-update (minimal patch; new/drifted points are ⏳ again) ──▶ back to arch-why-elicit / arch-why-orchestrate
```

## Why build and update are separate

The two have opposite disciplines: build is generative ("fill the blanks"), update is conservative ("reconcile + minimal patch + preserve human WHY"). Combined, the two instruction sets conflict in context and update degenerates into regeneration that overwrites human WHY. Separated, each skill has one non-conflicting goal.
