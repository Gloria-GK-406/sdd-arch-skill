---
name: arch-spec-flow
description: Use when entering or running any spec-driven development flow (e.g. brainstorm→spec→plan→impl→review). Fire regardless of repo or monorepo subproject, which area / plane / subsystem the change touches, or whether architecture-intent documents already exist there. Absence of docs in an area is a case this skill handles, never a reason to skip it.
---

<SUBAGENT-STOP>
If you are a subagent, ignore this skill's requirements. This is a conductor skill
for the main agent only; continue with the task you were explicitly dispatched to do.
</SUBAGENT-STOP>

# arch-spec-flow · Weave architecture intent into the spec-driven loop

## Overview

Spec-driven development (SDD) already has a shape: design → spec → plan → impl → review → merge. This skill pins architecture work to **two explicit SDD phases**. During the **spec phase**, the main agent reads the recorded architecture intent and records any deliberate boundary crossing in the spec; if no documents cover the footprint, the spec records that fact instead. During the **implementation phase**, after code has landed and tests / review pass, architecture review + document reconciliation run as the **final implementation step** before branch wrap-up / integration begins; for an uncovered footprint, skip review and go straight to reconciliation/build suggestions from landed code. You are the conductor that fits those phase steps into whatever SDD flow the main agent is running; the actual work is delegated to `arch-spec-review` / `arch-doc-reconcile` / `arch-doc-update` / `arch-doc-build` / `arch-why-elicit`.

**One iron principle governs everything**: **documents strictly trail code; they never lead it.** Design-time intent — including the intent to *cross* a recorded architecture boundary — lives in the **spec**, never prematurely in a document. A document is only ever written or updated **from landed code**, as the final implementation step on the branch, so every documented statement stays grounded in real code. This is why the spec-phase step declares intent in the spec, and the implementation-final step reconciles the document from landed code, rather than editing documents in the middle.

## When to use

- The main agent is running any spec-driven flow (writing a spec, or executing one to landing) — in **any repo or monorepo subproject**, **whether or not an architecture-intent tree already exists in the area being changed**. If none exists there yet, the implementation-final architecture step bootstraps it from the landed code; absence of docs is never a reason to skip this skill. (In a monorepo, "project alpha has docs, project beta doesn't" does **not** exempt beta — per-subproject doc state is irrelevant to whether this skill fires.)
- Run the spec-phase instructions **during the SDD spec phase**, before code is written. Run the implementation-phase architecture gate **after implementation code has landed and its tests / review pass**, as the **final step of SDD implementation**, before the host flow offers any merge / PR / integration choice. The implementation-final step is a **post-landing check gate**, not a step inside the merge decision: implementation is not complete until it clears.

**Do not use for**: a non-spec, one-off change with no spec artifact (run `arch-spec-review` directly if you want an architecture check); editing documents ahead of code (forbidden — see the iron principle).

## Iron Law

```
Documents trail code, never lead. No document is created or updated until the code it describes has landed.
The intent to cross a boundary lives in the spec (declared with doc-path + INV-id), not in the document.
The document is reconciled from landed code as the final implementation step ON THE BRANCH — code merges with documents already updated, never after.
```

## Architecture steps by SDD phase

### Spec phase — declare architecture intent before code is written

1. **Locate the footprint's documents** and load them as design aid: the architecture-intent documents covering the area this spec will change (resolve via `intent-contract` "Document location and naming"). Designing with the recorded intent in view is what prevents most accidental drift — do not skip it.
2. **Classify the spec against the intent** (using the `intent-contract` conventions to parse markers/invariants):
   - **Stays within intent** → no declaration needed; the implementation-final step will only need to record any genuinely new architectural surface.
   - **Deliberately crosses a marked boundary** (the spec's point is to move a `🧱/💡/🚧`) → **write an intent declaration into the spec** (format below). The architecture decision is the human's, made at **spec approval** — surfacing it here is what makes it conscious and reviewable.
   - **No document covers the footprint at all** → record that no architecture documents cover the footprint and that documents will be **bootstrapped from landed code** in the implementation-final step; declare nothing now.
3. **Never touch a document during the spec phase.** Intent goes into the spec only.

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

### Implementation phase final step — after code lands, before branch wrap-up

> **This is the final step of SDD implementation, not a step inside the merge decision.** It fires after the code is done and its tests / review pass — the same point at which the host flow might otherwise consider implementation complete and begin branch wrap-up. Implementation is not complete until this architecture review/reconcile or no-doc build sequence clears. If you are being asked *how* to merge before that final architecture step has run, the implementation phase was closed too early — back up and run it.

1. **Choose the route from the spec-phase record**:
   - **Relevant documents exist** → dispatch `arch-spec-review` (read-only compliance check), giving it as material: the diff, the relevant architecture documents, `intent-contract`, **and the spec document itself** (you know where this SDD flow keeps specs — pass that path; review does not go looking for it). The spec's intent declaration lets review run its consistency check (declared crossings realized? nothing undeclared crossed?). Review **changes nothing**; it returns a report — conflict artifacts for violations, reconciliation signals for compliant-but-document-affecting changes.
   - **The spec records that no documents cover the footprint** → **do not dispatch `arch-spec-review` for that footprint**. There is no recorded intent to check, so review would only report "cannot review"; skip directly to `arch-doc-reconcile` with the landed code and an explicit no-doc/no-review note so it can suggest the initial build.
2. **If review ran, resolve its violations before reconciling**: undeclared violations / inconsistencies → fix code or escalate per the conflict artifacts; a large deviation flagged by review → redesign the spec with the artifacts (do not blind-rerun). Re-review after a code fix.
3. **Dispatch `arch-doc-reconcile`** with review's report (or the explicit no-doc/no-review note), the architecture documents if any, the landed code, `intent-contract`, **and the spec** (it uses the spec's declarations to label which inconsistencies are expected vs unexpected). It is **read-only**: it classifies where the documents diverge from the landed code (per constraint point) and returns a **suggestion report** — per divergence, a suggestion (delete / update / add / create / migrate) and which atom fits, plus any architecture decision or undeclared inconsistency flagged for the human. It edits nothing and dispatches nothing.
4. **Present the suggestions to the author and adopt**: relay reconcile's report; the author adopts, amends, or rejects each suggestion (and settles any flagged architecture decision — a consciously moved boundary). This adoption is where authorization lives — in your hands and the author's, not inside reconcile.
5. **Carry out the adopted changes**: for each adopted suggestion run `arch-doc-update` (existing unit) or `arch-doc-build` (no document yet) to reconcile WHAT **from the landed code**; new or drifted points land as fresh `⏳`.
6. **Fill the WHY**: `arch-why-elicit` settles the WHY / strength for every fresh `⏳` — **on the branch, with the author present** — so the documents are `⏳`-free before merge. (If a WHY genuinely cannot be settled on-branch, the `⏳` lands and the **next** spec's review gate forces it — the strict gate is the safety net, not a license to skip.)
7. **Merge only after the documents are reconciled.** Code reaching main with stale or absent architecture documents is the exact window this flow exists to close.

**Dispatch `arch-spec-review` when applicable and always dispatch `arch-doc-reconcile` to the best available subagent / model** — both turn on subtle, high-stakes architectural judgment when they run; never run them on a cheap model. (Contrast the WHAT-writing atoms `arch-doc-update` / `arch-doc-build`, which carry out the adopted changes and may run cheaply, as `arch-doc-orchestrate` does.)

## Routing (footprint × intent), in one place

| Footprint has docs? | Spec's relation to intent | Spec phase | Implementation final step |
|---|---|---|---|
| yes | stays within intent | nothing to declare | review reports → `arch-doc-reconcile` suggests an update only if new architectural surface appeared → author adopts → `arch-doc-update` → `arch-why-elicit` |
| yes | deliberately crosses | declare crossing (doc-path + INV-id) | review consistency-checks the declaration → `arch-doc-reconcile` suggests the boundary move → author adopts (settles the decision) → `arch-doc-update` from landed code → `arch-why-elicit` |
| yes | accidentally crosses (caught designing) | redesign so it doesn't cross | — |
| no | (any) | record "no docs cover footprint; bootstrap in implementation final step" | skip `arch-spec-review` → `arch-doc-reconcile` suggests a build from landed code → author adopts → `arch-doc-build` from landed code → `arch-why-elicit` |

## Dispatching the review and reconcile subagents

- **Self-contained**: when review is in route, give the review dispatch the diff, the architecture documents, `intent-contract`, the `arch-spec-review` skill to follow, and **the spec document path**. Give the reconcile dispatch review's report **or the explicit no-doc/no-review note**, the architecture documents if any, the landed code, the spec, `intent-contract`, and the `arch-doc-reconcile` skill to follow. A subagent does not inherit your session.
- **The spec is optional to review, mandatory from you when review runs**: `arch-spec-review` works with or without a spec, but in an SDD flow you always have one — pass it, so review can distinguish intended crossings from accidental ones.
- **Best model for judgment**: dispatch review (when it runs) and reconcile to the **best available subagent / model** — their judgment is high-stakes. Do not run them cheaply.
- **Adoption stays with you and the author**: review and reconcile are both read-only and return reports. *You* relay reconcile's suggestions to the author, the author adopts/amends/rejects, and *you* run `arch-doc-update` / `arch-doc-build` on what was adopted. Neither subagent edits a document or dispatches an atom.

## Red flags — stop

- Editing or creating an architecture document **before** the code it describes has landed (violates the iron principle — intent belongs in the spec until then).
- A spec that crosses a marked boundary but carries **no intent declaration** (the crossing will read as accidental drift, correctly — but the author should have declared it; surface this).
- The host flow's branch wrap-up / integration step presenting merge / PR / cleanup options **before** the implementation-final architecture step has run. That step fires on "code landed + tests / review pass", and implementation is not complete until it clears — being asked *how* to merge before review/reconcile or no-doc build ran means the SDD phases were mis-ordered; back up.
- Reaching merge with documents not yet reconciled to the landed code.
- Treating reconcile's suggestion report as applied work — it only suggests; the documents are written by `arch-doc-update` / `arch-doc-build` after the author adopts.
- Letting `arch-doc-reconcile` (or `arch-spec-review`) edit a document or dispatch an atom — both are read-only.
- Running review or reconcile on a cheap model instead of the best available.
- Hardcoding the spec path into `arch-spec-review` instead of passing it from here.

## Self-check

- [ ] Spec-phase architecture step ran during the SDD spec phase: footprint docs loaded as aid when present; every deliberate boundary crossing declared in the spec with doc-path + INV-id; absent coverage recorded as no-doc; no document touched
- [ ] Implementation-final architecture step ran **after code landed (tests green) as the last step of SDD implementation, before any merge / PR option was offered**: if docs existed, `arch-spec-review` dispatched **with the spec document** as material and violations resolved; if the spec recorded no docs, review was skipped for that footprint
- [ ] `arch-doc-reconcile` dispatched with review's report or the no-doc/no-review note; it returned a **suggestion report** and edited nothing; suggestions relayed to the author and **adopted** before any document was written
- [ ] Review when applicable, and reconcile always, ran on the best available subagent / model
- [ ] Documents reconciled **from landed code** by `arch-doc-update` / `arch-doc-build` on the adopted suggestions, every fresh `⏳` cleared by `arch-why-elicit` on-branch
- [ ] Merge happened only after documents were reconciled
- [ ] No document was written or edited ahead of its code

## Dependencies / integration

- **Spec phase (declaration)**: parses documents per `../arch-docs-conventions/references/intent-contract.md`.
- **Implementation final step (review/reconcile or no-doc build path)**: dispatches `arch-spec-review` (with the spec) only when relevant documents exist, always dispatches `arch-doc-reconcile` with either review's report or the no-doc/no-review note — both read-only, both on the best available model when run — then relays reconcile's suggestions to the author, runs `arch-doc-update` / `arch-doc-build` on the adopted ones, and `arch-why-elicit` on the fresh `⏳`.
- **Tree-scale variants**: for a whole subsystem rather than one spec, the build/update side may run via `arch-doc-orchestrate` and the WHY side via `arch-why-orchestrate`.
- **Host flow**: this skill is an **add-on** to the project's existing spec-driven flow (e.g. `superpowers:writing-plans` / `superpowers:executing-plans` / `superpowers:finishing-a-development-branch`); it does not replace it. The implementation-final architecture step belongs to the host flow's implementation phase: it triggers when implementation has landed and tests / review pass, and must clear **before** branch wrap-up offers merge / PR / cleanup — the host's merge prompt fires only once implementation is complete.
