---
name: arch-doc-build
description: Use when a code unit (service / component / overview level) has no architecture document yet and one must be built from scratch; or when arch-doc-orchestrate dispatches you to build a document for a unit on the tree that has no document yet.
---

# arch-doc-build · Architecture document construction

## Overview

Given a code unit that has no architecture document yet, generate from scratch an isomorphic-document WHAT: analyze the code, split it into responsibility-disjoint subparts, fill in the "What" per the template, and leave every "Why" as `⏳ pending`.

Core principle: fill in only the WHAT (structural facts); leave WHY as `⏳` always. The rationale belongs to people and is supplied later by `arch-why-elicit`, after asking the people involved and grounding the answers in code. Violating the letter is violating the spirit: writing any "why / in order to / trade-off" into a quote block exceeds your scope.

## When to use

- Build an architecture document that does not yet exist for a unit (overview level or component level).
- When dispatched by `arch-doc-orchestrate` to build a document for a unit on the tree that has no document yet.

Do not use for: a document that already exists (use `arch-doc-update`); filling in WHY (use `arch-why-elicit`); reviewing a landed spec (use `arch-spec-review`).

## Iron Law

```
Fill in only WHAT. Every why quote block is `⏳ pending`. Write not one word of rationale.
```

Code comments, commits, and your own inferences are full of ready-made rationale — none of them is a legitimate source of WHY (a comment may itself be a drift artifact). When you see a tempting "because / in order to," record only its resulting fact in the WHAT and leave the "why" to `⏳`.

## Workflow

1. Read the target unit's code closely. List the directory (use PowerShell `Get-ChildItem -Recurse` when paths contain spaces or non-ASCII characters, since globs fail easily), then read the key files (assembly entry, external surface, coordination / orchestration, core execution, shared state).
2. Decompose into subparts, and decide for each whether it goes on the main axis or into the drill-down section (criteria below). Include the hub / shared state that runs throughout, and the composition root assembly entry.
3. Declare the `<unit>` code. Derive a slug from the unit name / path and write it into the preamble (immediately after ①, before ②, on its own line). It is the id prefix for this document.
4. Fill in the WHAT per `template.md`: the three preamble blocks + §1 structural overview (ASCII diagram + three-column table) + §3..N per-subpart sections (open with `**What**:` + table + `→ code: path:line`) + the drill-down section. Delete the template's fill-in guidance comments and the `<details>` example.
5. Write boundary cells as atomic dual-state entries: one atomic statement per line, format `⏳[INV-<unit>-<slug>] <statement>` (since you are build, the strength-marker slot is always `⏳`; do not mark 🧱/💡/🚧).
6. Write every why block as `> **Rationale** ⏳ pending` — leave only the placeholder.
7. Drill-down section: for collaborating components give only "component + seam + one-line responsibility + dedicated-document pointer"; do not mint ids for them. Set each pointer to one of three states — a **link** (exists), `to-be-written` (warrants its own document, not yet written), or `—` (a one-line row suffices; no document). See "Granularity: which components warrant a document" below.
8. Run the output self-check (see end), and prompt the people involved to run `arch-why-elicit` afterward to fill in WHY.

**Output location**: by default write to `docs/architecture/[<project>/]<unit-path>/00-<name>.md` for an overview (per `../arch-docs-conventions/references/intent-contract.md`, "Document location and naming"). When dispatched by `arch-doc-orchestrate`, use the path it gives.

## Key judgments

Apply criteria rather than intuition.

### Main axis vs drill-down

| Goes on the main axis (its own §section) | Goes into the drill-down section (mark seam + pointer only) |
|---|---|
| Non-overlapping skeleton segments of "one execution" at the top level; real control-flow turns / seams | New'd by the composition root, injected through a seam, or called, but internally self-contained |
| The hub / shared state that runs throughout | Has an independent rate of change and a unit-testable boundary, and could each become its own isomorphic lower-level document |
| The composition root assembly entry | In the main axis, needs only "one-line responsibility + seam" |

When unsure, ask: "If this document expands it, does the reader understand the main axis better, or get drowned in detail?" — the latter means it sinks into the drill-down section.

### Granularity: which components warrant a document

Not every drill-down component needs its own document. Set the dedicated-document pointer to:
- `to-be-written` — when the component is self-contained, has an independent rate of change, and is **complex enough that the one-line row cannot capture it** (it has internal structure worth its own isomorphic document). This is a decomposition signal: orchestrate will build it, and the tree recurses.
- `—` — when **a one-line responsibility is enough** (a small adapter, a thin helper). It stays a permanent leaf; no document is built.

When unsure, prefer `—`: an under-documented component is cheap to promote later (add `to-be-written`), while an over-split tree sprawls. Do not mark `to-be-written` just because a component exists — mark it only when it earns a document.

### Boundary cells: what qualifies, how fine to slice

- Qualify before including: a boundary is an architecturally meaningful constraint (what this component must not know / touch / do; hard cross-layer boundaries), not every defensive code path or implementation detail.
- Atomic granularity: one entry = one independently violable, independently up/downgradable assertion. Do not leave it un-split, and do not merge two items whose strength may differ (e.g. "does not touch the registry" may be 🧱 while "does not send HTTP yet" may be 🚧 — they must be on separate lines).
- When unsure whether something is intent: write it as `⏳` so it can be decided or removed during upgrade — but do not stuff the boundary cell with low-value assertions and burden manual review.

<Good>
```
⏳[INV-exec-no-registry] Does not touch the registry <br> ⏳[INV-exec-no-http] Does not send HTTP yet
```
Two independent assertions, one id each, upgradable to different strengths separately.
</Good>

<Bad>
```
⏳[INV-exec-isolated] Does not touch registry / persistence / HTTP; pure execution
```
Three things crammed into one entry, one id, strength forced to a single value; when review reports a "violation" you cannot tell which one was broken.
</Bad>

### Depth and numbering

- Load-bearing mechanisms may be expanded: the most critical stretch of a flow (e.g. a synchronous precheck sequence) may be expanded as a numbered sequence (the §N shape in the template); do not compress it into a single boundary-cell line. Keep the other subparts' tables concise.
- Sequence / narrative sections: number them by the actual ordinal (e.g. `## 7.`), not the literal `N` / last used in the template; narrative sections mint no new ids (they only thread together existing subparts).

## Rationalizations

| Rationalization | Reality |
|---|---|
| "The code comment already states the reason; copy it into the why block to save effort." | A comment is not the designer's answer under questioning and may itself be drifting. Take only the resulting fact into the WHAT and leave the rationale as ⏳. |
| "This reason is too obvious; filling it in is harmless." | Whether it is obvious is for a person to judge. An "obvious reason" you fill in impersonates a human source and pollutes the true source of WHY. |
| "Explaining in one line why the boundary is drawn this way is clearer." | That is WHY. `What` records only structural facts; all explanation goes into the `⏳` quote block. |
| "This one is 🧱; I'm fairly sure." | Marking strength is a person's job. You leave only `⏳`; let `arch-why-elicit` set the strength after grounding it with a person. |

## Red flags — stop

- Any why quote block contains "because / in order to / trade-off / chose X over Y"
- A boundary cell contains 🧱/💡/🚧 (you should write only `⏳`)
- A reason from a code comment / commit was moved into the document
- An id was minted for a collaborating component in the drill-down section
- Code was changed to "tidy it up," or behavior not present in the code was inferred

Any of these = you exceeded scope by writing WHY or changing the current state. Delete it and revert to filling in only WHAT.

## Output self-check

- [ ] Every why quote block is `⏳ pending`, with no word of rationale
- [ ] Every boundary is `⏳[INV-<unit>-<slug>] <statement>`, atomic, with no strength marker
- [ ] `<unit>` is declared in the preamble, all ids use it as prefix, semantic slug rather than serial number
- [ ] Every `→ code: path:line` points to a real file and line number
- [ ] The main-axis / drill-down split has criteria; drill-down rows have no id
- [ ] Narrative / sequence sections mint no new ids; section numbers are the real ordinals
- [ ] Faithfully reflects the current code; nothing inferred, tidied, or copied from any existing document; no business code changed

## Dependencies

`../arch-docs-conventions/assets/template.md` · `../arch-docs-conventions/references/intent-contract.md`
