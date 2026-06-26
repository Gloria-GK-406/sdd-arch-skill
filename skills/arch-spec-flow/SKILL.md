---
name: arch-spec-flow
description: Use when doing spec-driven development (writing or executing a spec, e.g. a brainstorm→spec→plan→execute→review flow) in a project that maintains architecture-intent documents (an arch-docs-conventions tree, by default under docs/architecture/). Applies whether or not such documents already exist for the area being changed.
---

# arch-spec-flow · Weave architecture intent into the spec-driven loop

## Overview

Spec-driven development (SDD) already has a shape: design → spec → plan → execute → review → merge. This skill adds **two architecture touchpoints** to that shape — one at spec finalization, one at branch wrap-up — and **dispatches the existing `arch-skills` at the right moments**. You are the conductor that splices the architecture steps into whatever SDD flow the main agent is running; the actual work is delegated to `arch-doc-build` / `arch-doc-update` / `arch-why-elicit` / `arch-spec-review`.

**One iron principle governs everything**: **documents strictly trail code; they never lead it.** Design-time intent — including the intent to *cross* a recorded architecture boundary — lives in the **spec**, never prematurely in a document. A document is only ever written or updated **from landed code**, as the last step on the branch, so every documented statement stays grounded in real code. This is why there is a front touchpoint (declare intent in the spec) and a back touchpoint (reconcile the document from the landed code) rather than one doc-editing step in the middle.

## When to use

- The main agent is running any spec-driven flow (writing a spec, or executing one to landing) in a repo that keeps an architecture-intent tree.
- Invoke it **once at spec finalization** (touchpoint A) and **once the moment the implementation lands and tests pass** — *before* the host flow offers any merge / PR / integration choice (touchpoint B). Touchpoint B shares its trigger with `superpowers:finishing-a-development-branch` ("implementation complete, all tests pass") and must run **first**, gating it — it is a post-landing check gate, not a step inside the merge decision.

**Do not use for**: a non-spec, one-off change with no spec artifact (run `arch-spec-review` directly if you want an architecture check); editing documents ahead of code (forbidden — see the iron principle).

## Iron Law

```
Documents trail code, never lead. No document is created or updated until the code it describes has landed.
The intent to cross a boundary lives in the spec (declared with doc-path + INV-id), not in the document.
The document is reconciled from landed code as the final step ON THE BRANCH — code merges with documents already updated, never after.
```

## The two touchpoints

### Touchpoint A — at spec finalization (before code is written)

1. **Locate the footprint's documents** and load them as design aid: the architecture-intent documents covering the area this spec will change (resolve via `intent-contract` "Document location and naming"). Designing with the recorded intent in view is what prevents most accidental drift — do not skip it.
2. **Classify the spec against the intent** (using the `intent-contract` conventions to parse markers/invariants):
   - **Stays within intent** → no declaration needed; the back touchpoint will only need to record any genuinely new architectural surface.
   - **Deliberately crosses a marked boundary** (the spec's point is to move a `🧱/💡/🚧`) → **write an intent declaration into the spec** (format below). The architecture decision is the human's, made at **spec approval** — surfacing it here is what makes it conscious and reviewable.
   - **No document covers the footprint at all** → note that documents will be **bootstrapped from landed code** at touchpoint B; declare nothing now.
3. **Never touch a document in touchpoint A.** Intent goes into the spec only.

**Intent-declaration format** (a section in the spec; one block per crossing):
```
## Architecture impact (intent declaration)
- doc:       docs/architecture/execution-plane/00-main-structure.md
  invariant: INV-exec-core-complete-contract
  current:   🧱 success = result.json:complete && verdict.ok
  intended:  relax to complete-only on the fast path
  why:       <reason the boundary should move>
```
The **doc path + INV-id** is the precise anchor `arch-spec-review` uses to tell an *intended* crossing from an *accidental* one.

### Touchpoint B — the moment code lands (implementation complete, tests green), before any merge / PR decision

> **This is a post-landing gate, not a step inside the merge decision.** It fires the instant the code is done and tests pass — the *same* trigger as `superpowers:finishing-a-development-branch`. When the host flow is about to offer merge / PR / cleanup options, touchpoint B runs **first** and the integration decision waits behind it. If you are being asked *how* to merge before review + reconcile have run, the gate has already been skipped — back up and run it.

1. **Dispatch `arch-spec-review`**, giving it as material: the diff, the relevant architecture documents, `intent-contract`, **and the spec document itself** (you know where this SDD flow keeps specs — pass that path; review does not go looking for it). The spec's intent declaration lets review run its consistency check (declared crossings realized? nothing undeclared crossed?).
2. **Resolve review's findings**: undeclared violations / inconsistencies → fix code or escalate per the conflict artifacts; a large deviation → redesign the spec with the artifacts (do not blind-rerun).
3. **Reconcile the documents from the landed code**:
   - footprint **had documents** → `arch-doc-update` brings WHAT in line with the landed reality (including any boundary consciously moved); new architectural surface gets new `⏳` invariants.
   - footprint **had no document** → `arch-doc-build` bootstraps it from the landed code.
   - then `arch-why-elicit` fills the WHY / strength for every fresh `⏳` — **on the branch, with the author present** — so the documents are `⏳`-free before merge. (If a WHY genuinely cannot be settled on-branch, the `⏳` lands and the **next** spec's review gate forces it — the strict gate is the safety net, not a license to skip.)
4. **Merge only after the documents are reconciled.** Code reaching main with stale or absent architecture documents is the exact window this flow exists to close.

## Routing (footprint × intent), in one place

| Footprint has docs? | Spec's relation to intent | Touchpoint A | Touchpoint B (after review) |
|---|---|---|---|
| yes | stays within intent | nothing to declare | update only if new architectural surface appeared |
| yes | deliberately crosses | declare crossing (doc-path + INV-id) | review consistency-checks the declaration → `arch-doc-update` moves the boundary from landed code → `arch-why-elicit` |
| yes | accidentally crosses (caught designing) | redesign so it doesn't cross | — |
| no | (any) | note "bootstrap at B" | `arch-doc-build` from landed code → `arch-why-elicit` |

## Dispatching the review subagent

- **Self-contained**: give the review dispatch the diff, the architecture documents, `intent-contract`, the `arch-spec-review` skill to follow, and **the spec document path**. A subagent does not inherit your session.
- **The spec is optional to review, mandatory from you**: `arch-spec-review` works with or without a spec, but in an SDD flow you always have one — pass it, so review can distinguish intended crossings from accidental ones.

## Red flags — stop

- Editing or creating an architecture document **before** the code it describes has landed (violates the iron principle — intent belongs in the spec until then).
- A spec that crosses a marked boundary but carries **no intent declaration** (the crossing will read as accidental drift, correctly — but the author should have declared it; surface this).
- The host flow's integration step (e.g. `superpowers:finishing-a-development-branch`) presenting merge / PR / cleanup options **before** touchpoint B has run. Touchpoint B fires on "code landed + tests green" and the integration decision waits behind it — being asked *how* to merge before review + reconcile ran means the gate was mis-ordered; back up.
- Reaching merge with documents not yet reconciled to the landed code.
- Doing the doc work yourself in the conductor instead of dispatching `arch-doc-update` / `arch-doc-build` / `arch-why-elicit`.
- Hardcoding the spec path into `arch-spec-review` instead of passing it from here.

## Self-check

- [ ] Touchpoint A ran at spec finalization: footprint docs loaded as aid; every deliberate boundary crossing declared in the spec with doc-path + INV-id; no document touched
- [ ] Touchpoint B ran **the moment code landed (tests green), before any merge / PR option was offered**: `arch-spec-review` dispatched **with the spec document** as material; findings resolved
- [ ] Documents reconciled **from landed code** (update or build), every fresh `⏳` cleared by `arch-why-elicit` on-branch
- [ ] Merge happened only after documents were reconciled
- [ ] No document was written or edited ahead of its code

## Dependencies / integration

- **Front (declaration)**: parses documents per `../arch-docs-conventions/references/intent-contract.md`.
- **Back (review + reconcile)**: dispatches `arch-spec-review` (with the spec) → `arch-doc-update` or `arch-doc-build` → `arch-why-elicit`.
- **Tree-scale variants**: for a whole subsystem rather than one spec, the build/update side may run via `arch-doc-orchestrate` and the WHY side via `arch-why-orchestrate`.
- **Host flow**: this skill is an **add-on** to the project's existing spec-driven flow (e.g. `superpowers:writing-plans` / `superpowers:executing-plans` / `superpowers:finishing-a-development-branch`); it does not replace it. Touchpoint B **precedes** the host's integration step: it triggers on the same "implementation complete, tests pass" condition as `superpowers:finishing-a-development-branch` and must clear **before** that step offers merge / PR / cleanup — the host's merge prompt fires only once touchpoint B has passed.
