---
name: arch-doc-reconcile
description: Use when code has landed in some scope and you need a concrete list of suggested architecture-document changes. Compares the changed code against the current documents at constraint-point (invariant) granularity, classifies the divergences, and produces a suggestion report. Read-only — writes no document, dispatches nothing, decides nothing. The suggestions are adopted by a human and carried out by arch-doc-build or arch-doc-update.
---

# arch-doc-reconcile · Document reconciliation suggestions

## Overview

Given the code changes in a defined scope and the current architecture documents, compare them **constraint point by constraint point** (each documented boundary invariant `INV-…`) and at the **document-tree placement** level, classify where they have diverged, and produce a **suggestion report**: the document changes that would bring the documents back in line, each naming the atom (`arch-doc-update` / `arch-doc-build`) that would carry it.

The skill's entire output is the report. It **modifies nothing** — no document, no code — and **dispatches nothing**. Whoever runs it presents the suggestions to the developer, who adopts, amends, or rejects them; the writing is then done by `arch-doc-build` / `arch-doc-update` from the landed code, and any new `⏳` is filled by `arch-why-elicit`.

**Core principle**: you are an analyst producing advice, not an actor. Detect the divergence, classify it, suggest the change, name the atom — as a recommendation. You write no document, dispatch no subagent, and make no architecture decision.

## When to use

- After a change lands in some scope and the documents may lag the code — to get a concrete, reviewable list of suggested document changes before anyone edits a document.
- In `arch-spec-flow` touchpoint B, after `arch-spec-review`, to turn the code↔document divergence into adoptable suggestions.

**Do not use for**: writing the documents (`arch-doc-build` / `arch-doc-update`), the compliance check (`arch-spec-review`), filling WHY (`arch-why-elicit`), or batch-building a whole WHAT tree (`arch-doc-orchestrate`).

## Iron Law

```
Produce suggestions only. Modify nothing — no document, no code. Dispatch nothing, decide nothing.
Compare the changed code to the documents per constraint point; classify each divergence; suggest the change and name the atom.
Architecture decisions (move a 🧱, accept a relaxation, rewrite a rationale) and undeclared inconsistencies are flagged for the human, never resolved here.
WHY and strength markers are never written — a suggested new point is noted as a future ⏳ for arch-why-elicit.
```

## Workflow

1. **Ingest**: read the changed code/diff in the scope, the current architecture documents covering that scope (resolve via `intent-contract` "Document location and naming"), `intent-contract` itself, and — when provided — the spec (its declarations classify expected vs unexpected inconsistency) and `arch-spec-review`'s report (strong leads, not a substitute for your own reading).
2. **Detect divergence** across the scope — the five kinds below, judged per constraint point and per document-tree placement.
3. **Classify each divergence into a suggestion** — the five kinds below — naming the target document/section and the atom(s) that fit.
4. **Handle inconsistencies (detection 2) by spec**: with a spec, label each as **expected** (the spec declared it, by doc-path + INV-id → suggest the update) or **unexpected** (not declared → surface it and flag it for the human, do not blindly suggest editing the document to match the code). Without a spec, surface every inconsistency without judging intended-vs-accidental.
5. **Flag architecture decisions** (a 🧱 to move, a relaxation to accept, a rationale that may be wrong, any unexpected inconsistency) as **human decisions**, not as suggested edits.
6. **Assemble the report** (shape below) and return it. Edit nothing.

## Detection — the five divergences

Judged within the change scope, against the current documents:

1. **Vanished constraint** — the document records an `INV-…` the landed code no longer enforces (the constraint was removed).
2. **Inconsistent constraint** — the document records an `INV-…` but the landed code contradicts its statement (implementation ≠ recorded constraint). A sub-flavor is **anchor drift**: the constraint's core still holds but the WHAT symbol/structure it names moved with the code.
3. **Unrecorded constraint** — the landed code enforces an architecturally meaningful constraint the documents never recorded. **Sub-judgment**: does it belong **under an existing document** (a new invariant in a unit that already has a document), or does it **warrant a new document** (an uncovered footprint / a unit with no document)? Apply the same granularity test as build (`intent-contract` §5 — self-contained, independent rate of change, complex enough to need its own isomorphic document).
4. **Migrated constraint (whole-component)** — a component moved or was renamed at the component level, so the document's path no longer tracks the component's code path — **doc-path ↔ component-path drift**. The constraints' meaning is unchanged; their carrying document must move and the drill-down pointers re-align.
5. **Split-out constraint** — a component grew until the portion of the current document carrying it has reached the threshold of its own isomorphic document (`intent-contract` §5 — a `—` row or an inline part now warrants `to-be-written` / its own file). The constraints must move out into a newly split file.

## Suggestions — the five proposals

Each maps a divergence to a concrete document change, naming the atom that would carry it:

| Suggestion | For detection | What it does | Atom(s) |
|---|---|---|---|
| **Delete a constraint** | 1 vanished | remove the `INV-…` line together with its id (no tombstone) | `arch-doc-update` |
| **Update a constraint** | 2 inconsistent | restate the invariant's statement / re-anchor it to match the landed code, **id stable**; if human WHY is anchored to the old statement, note that it drops to `⏳` for re-confirmation | `arch-doc-update` |
| **Add a constraint** | 3 → under existing doc | add a new invariant (born `⏳`) into an existing document/section | `arch-doc-update` |
| **Create a constraint** | 3 → needs new doc | create a **new document** that carries the new invariant (born `⏳`); the parent's drill-down pointer becomes a link | `arch-doc-build` (+ `arch-doc-update` on the parent) |
| **Migrate a constraint** | 4 component migration · 5 split-out | move the constraints (and/or their document) to track the code — for 4, move/rename the document to the new component path and re-align drill-down pointers; for 5, split the carrying portion into a new file and re-point the parent (`to-be-written` → link) | `arch-doc-update` (± `arch-doc-build` for the new file) |

**Update is not delete-plus-add.** A diverged constraint whose identity persists is an **update** (id stable). Delete + add appear together only when a constraint genuinely **vanished** and a **different new** one appeared — two separate detections (1 and 3), not one inconsistency.

## Key judgments

### Suggest, never act

Your output is advice. You never edit a document, never run `arch-doc-build` / `arch-doc-update`, never start an interview. The developer decides what to adopt; the atoms do the writing. A suggestion that reads as "here is what I changed" is wrong — it is "here is what I suggest changing".

### Inconsistency: surface it, classify it against the spec, but do not launder a violation

When the code contradicts a recorded constraint, the disposition depends on the spec. **With a spec**, an inconsistency the spec **declared** is an expected modification → suggest the update that tracks it. An inconsistency the spec did **not** declare is unexpected → surface it and flag it for the human; do **not** suggest editing the document to match the code, because that would silently bless an undeclared crossing as if it were intended. **Without a spec**, you cannot tell intended from accidental — surface every inconsistency and let the human supply the intent.

### Add vs create: whether a new document is born

The only difference between an **add** and a **create** is whether a new document file is born. The constraint goes under an existing document → add (an `arch-doc-update`). The footprint warrants its own isomorphic document → create (an `arch-doc-build`, plus re-pointing the parent's drill-down). Apply build's granularity test; when unsure, prefer add — promoting later is cheap, an over-split tree sprawls.

### Migration is a placement change, not a content change

Detections 4 and 5 do not change what a constraint *means*; they change *where it lives* so the document tree keeps tracking the code (the `NN-` prefix is reorderable, identity is the `<unit>` code and `INV-` id). A migration suggestion preserves invariant ids and re-aligns drill-down pointers; it is not an excuse to reword constraints.

### WHAT divergence only; WHY is downstream

You reconcile structural facts against code. A suggested new or re-anchored point is noted as a future `⏳`; you never suggest a rationale or a strength marker — those are the human's, via `arch-why-elicit`.

## Output (suggestion report)

A list of suggestions plus any flagged human-decisions. Per suggestion:
```
detection       : inconsistent constraint (anchor drift)
document        : docs/architecture/execution-plane/00-main-structure.md  §3
constraint      : INV-exec-core-complete-contract
divergence      : WHAT describes runTask; code refactored to runEpisode
suggestion      : update — re-anchor §3 to runEpisode; old WHY drops to ⏳ for re-confirmation
atom            : arch-doc-update
spec            : expected (declared INV-exec-core-complete-contract)   | unexpected | n/a (no spec)
human-decision? : no
```
Use `suggestion: delete | update | add | create | migrate`. A create names the new document path and the parent to re-point. A migrate names the source and target paths. Unexpected inconsistencies and architecture decisions carry `human-decision?: yes` with the choice spelled out. The report edits nothing; it is consumed by the conductor / developer.

## Rationalizations

| Rationalization | Reality |
|---|---|
| "The change is obvious, I'll just edit the doc" | You produce suggestions, not edits. Modify nothing; the developer adopts and the atoms write. |
| "I'll run `arch-doc-update` myself to save a step" | Dispatching is not your job. Name the atom in the suggestion; the flow runs it after adoption. |
| "Doc and code disagree, so I'll suggest rewriting the doc to match the code" | Only if the spec declared it (expected). An undeclared inconsistency may be a violation — flag it for the human; do not launder it into the document. |
| "The constraint's wording changed, so delete the old and add a new one" | A persisting constraint is an **update** with a stable id. Delete + add is only for a constraint that truly vanished plus a different new one. |
| "This new boundary is clearly a 💡" | Strength markers and WHY are the human's, via `arch-why-elicit`. Note it as a future `⏳`. |
| "The artifact says a 🧱 should move, so I'll suggest the moved boundary as the edit" | Moving a 🧱 is the human's decision. Flag it as a decision; do not pre-bake it into a suggested edit. |

## Red flags — stop

- Edited a document or any code (this skill is read-only — its output is a report).
- Dispatched `arch-doc-update` / `arch-doc-build` / `arch-why-elicit` (you name the atom; you do not run it).
- Suggested editing a document to match an **undeclared** code change (laundering a possible violation) instead of flagging it.
- Expressed a persisting constraint's drift as delete + add instead of an update (breaking the stable id).
- Wrote a rationale or assigned a strength marker (the human's, via `arch-why-elicit`).
- Baked an architecture decision (a moved 🧱, an accepted relaxation) into a suggested edit instead of flagging it.

**Any of these = you acted instead of advising, or laundered a decision. Back out: detect, classify, suggest, name the atom, and leave everything unedited.**

## Self-check

- [ ] Output is a suggestion report only; no document and no code modified
- [ ] Each divergence classified into one of the five detections, and each suggestion is one of delete / update / add / create / migrate, naming the document and atom
- [ ] Inconsistencies labeled expected / unexpected against the spec (or surfaced plainly when no spec); unexpected ones flagged for the human, not turned into edits
- [ ] Persisting constraints suggested as updates with stable ids; delete + add reserved for a genuine vanish + new appearance
- [ ] No subagent dispatched; no atom run; no WHY written, no strength marker assigned (new points noted as future `⏳`)
- [ ] Architecture decisions flagged as human-decisions, not baked into edits

## Dependencies / integration

- **Inputs**: the changed code/diff in scope · the architecture documents covering the scope · `../arch-docs-conventions/references/intent-contract.md` (divergence judged per its conventions — boundaries/ids §3, stable id §4, seam/drill-down and placement §5, footprint §6, anchor drift) · optionally the spec (classifies expected vs unexpected inconsistency) · optionally `arch-spec-review`'s report.
- **Consumed by**: the conductor / developer, who adopt suggestions and then run `arch-doc-update` / `arch-doc-build`; new `⏳` is filled by `arch-why-elicit`.
- **Conductor**: `arch-spec-flow` runs this at touchpoint B and presents its suggestions to the developer.
