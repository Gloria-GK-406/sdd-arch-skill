---
name: arch-doc-update
description: Use when a unit of an existing architecture document has changed architecturally due to an implementation change and the document must be reconciled against the new code; or when the spec flow runs an update after a developer adopts an arch-doc-reconcile suggestion, or when dispatched by arch-doc-orchestrate to update a unit on the tree that already has a document but whose code has changed.
---

# arch-doc-update · Architecture document update

## Overview

Given an **existing document** plus **changed code**, perform a **minimal-patch** update: update only the WHAT of affected sections, leave unchanged sections completely untouched, preserve human-written WHY verbatim, and mark newly appearing decision points with `⏳`.

**Core principle**: you are a **reconciler and patcher**, not a rewriter. **Minimal change**: zero edits to unchanged sections, not a single character touched in human-written WHY. Violating the letter is violating the spirit: regenerating a section that did not actually change, or adjusting old WHY "to read better," are both overreach.

**But "minimal" does not mean "additive only" — additions, edits, and deletions all need to happen.** The document records the **present**, not history: when code removes a structure or constraint, **delete it cleanly** from the document (no tombstone, no "formerly such" comment). Additive bias has two faces, both adversaries: (1) editing too much (rewriting unchanged sections, touching WHY); (2) deleting too little (only adding the new while deleting nothing for what has disappeared, so the document grows longer and drifts further from the code). A healthy update means deleting what should be deleted, editing what should be edited, and adding what should be added.

## When to use

- When the spec flow runs an update after a developer **adopts** an `arch-doc-reconcile` suggestion (reconcile only suggests — it does not dispatch you; the flow runs you on the adopted change).
- When dispatched by `arch-doc-orchestrate` to update a unit on the tree that already has a document but whose code has changed.

**Do not use for**: a document that does not exist (use `arch-doc-build`); filling in WHY (use `arch-why-elicit`); reviewing a spec (use `arch-spec-review`).

## Iron Law

```
Minimal patch. Unchanged sections: not a single character changed. Human-written WHY: not a single word touched.
Existing ids never change; never regenerate, never renumber, never assign strength markers on someone's behalf.
```

## Workflow

1. **Reconcile**: existing document vs. current code; compute the **delta** section by section — which sections' WHAT changed, which did not, which subparts were added or removed.
2. **Update only the WHAT of affected sections**: patch the section where the change lands; leave unchanged sections **exactly as-is** (including their boundaries, ids, WHY, and code pointers).
3. **Preserve human-written WHY**: do **not touch a single word** of filled-in why blocks (unless "anchor drift" below is triggered → even then only **mark**, do not rewrite).
4. **Mark new/changed decision points with `⏳`**: mint a **new** id for a newly appearing boundary (`INV-<unit>-<slug>`, reusing this document's `<unit>`), set the strength slot to `⏳`; write `⏳ pending` for a newly added why block.
5. **Stable ids + deletion is deletion**: existing invariants **keep their original id** (even when the statement is reworded); new boundaries get new ids; **for a constraint the code removed, delete the boundary line together with its id, leaving no tombstone and no comment residue** — the document records the present, history belongs to git.
6. **Sync drill-down pointers**: a new subcomponent → add a line (dedicated doc `to write`); a removed one → **delete the line**; rename/migration → update the pointer and align `<unit>`.
7. **Run the self-check** (see end).

## Key judgments

### "WHAT restatement" ≠ "regeneration" — do not be tripped up by the Iron Law

The Iron Law says "never regenerate," but when a section's code **genuinely changes substantially** (e.g., the convergence model is replaced wholesale), its WHAT must be **restated at length** to fit the code — this is **not** the forbidden "regeneration." Keep the two distinct:

- **Allowed and required**: for a **changed section**, restate the structural facts per the new code (ASCII diagram, table rows, `What`, code pointers). WHAT belongs to the skill and may be freely rewritten to track the code.
- **Forbidden "regeneration"**: rewriting a section whose **code did not change**; or rewriting **any WHY prose**.

In one line: **the minimality constraint applies to the "load-bearing walls" — WHY prose, existing ids, section numbers, unchanged sections; WHAT itself is not subject to minimality.** Even a full WHAT refresh of a section is a compliant update, not a regeneration, as long as the load-bearing walls are untouched.

### "Which section counts as changed" — judge by WHAT, not by line number

A section "changed" = its **structural facts** (subpart responsibilities, boundaries, seams, code anchors) no longer match the code. A mere line-number shift or equivalent wording **does not count as changed**; do not rewrite for it. Test: if you follow this section's `→ code: path:line` and the structure and text still line up, it is **unchanged**.

### Anchor drift: WHAT changed, but the old human-written WHY still points at the old structure

This is the most critical case in update. A section's WHAT changed (e.g., `runTask`→`runEpisode`), and it has a **human-written WHY that still references the old structure**. You may **neither rewrite that WHY** (it belongs to a human) **nor pretend it still holds** and leave it unaddressed.

**Procedure**: update this section's WHAT, and **place a hand-off marker** on that WHY — **keep the old WHY prose verbatim** (as context for human re-review), **lower its strength slot back to `⏳`**, and immediately follow with an HTML comment line `<!-- anchor drift: WHAT updated (…), original WHY still anchored on (old symbol), pending human confirmation of migration -->`. **You only detect and mark; you do not adjudicate or rewrite prose.**

**This is the hand-off to `arch-why-elicit`**: lowering to `⏳` → the completeness gate therefore halts; the next time why-elicit scans a `⏳` carrying an "anchor drift" comment, the question shifts from "why" to **"can the old reasoning migrate to the new structure."** Update only detects, why-elicit migrates; they mesh via "⏳ + comment + preserved old prose."

**One symbol's drift affecting WHY in multiple sections** (e.g., `runTask`→`runEpisode` hitting the WHY in both §1 and §7): **each affected WHY block is an independent slot, each independently lowered to `⏳` + marked** — do not mark only one.

<Good>
WHAT updated from `runTask` to `runEpisode`; the old WHY block is preserved verbatim, but its strength is lowered from `💡` to `⏳`, with an added line `<!-- anchor drift: WHAT updated, original WHY points at runTask, pending human confirmation of migration -->`.
</Good>

<Bad>
Directly changing `runTask` to `runEpisode` in the old WHY and polishing it along the way — this impersonates a human and rewrites the true source of the WHY.
</Bad>

### Boundaries of the minimal patch

- A section only partially changed → touch only the changed lines/boundaries, leave the rest as-is.
- Do not touch any unchanged content for "style consistency" or "smoother reading."
- Do not renumber sections, do not re-mint unchanged ids.

## Rationalizations

| Rationalization | Reality |
|---|---|
| "Regenerating the whole section is easier and more consistent" | Regeneration wipes out human-written WHY, edits unchanged content, and creates churn. Patch only the changed lines. |
| "The `runTask` mentioned in the old WHY is gone, so I'll update it to the new name" | That rewrites the true source of the WHY. Update the WHAT; only **mark** the WHY as anchor drift and hand it off. |
| "This WHY reads awkwardly, let me polish it" | Smoothness is not your call. Human-written WHY: not a single word touched. |
| "The ids look messy, let me renumber them all" | Existing ids never change and are not renumbered — an id is the stable anchor of the same assertion across updates; renumbering breaks references. |
| "This section didn't change, but the wording could be better" | Unchanged means zero edits. You are not here to optimize prose style. |
| "I'm fairly sure this new boundary is 🧱, let me mark it" | Assigning strength is a human's job. New boundaries are always `⏳`. |
| "It's nice to leave a tombstone/comment as history for what the code removed" | The document records the present, not history. Delete cleanly; history belongs to git. Leaving residue = additive bias; the document only grows longer and worse. |
| "I only added the new and edited the changed, I didn't delete anything" | That very likely means you missed a deletion. Structures/constraints the code removed must be deleted from the document — deleting too little is as much a defect as editing too much. |

## Red flags — stop

- Rewrote or polished any human-written WHY block
- Regenerated a section whose code did not actually change
- Changed an existing `INV-` id, or renumbered sections
- Marked a new boundary 🧱/💡/🚧 (it should be `⏳`)
- The diff touched a section whose code did not change
- Wrote history residue such as "tombstone / deleted / formerly such"
- This update **deleted nothing** — while the code clearly removed some structures/constraints (deleting too little = additive bias)

**The first 5 = editing too much/overreach; the last 2 = deleting too little/additive bias. Roll back both faces: delete what should be deleted, leave unchanged content untouched, do not touch WHY/ids.**

## Output self-check

- [ ] Unchanged sections are **verbatim untouched** (including boundaries, ids, WHY, code pointers)
- [ ] Human-written WHY is unchanged; anchor-drift WHY is only **marked**, not rewritten
- [ ] New boundaries got new ids with strength `⏳`; all existing ids are preserved, not renumbered
- [ ] Structures/constraints the code removed have had their boundary lines and ids **deleted cleanly** (no tombstone, no leftover comments)
- [ ] Drill-down pointers in the final section align with the current subcomponents (additions/**removals**/renames all synced)
- [ ] No strength markers assigned on someone's behalf, no business code changed

## Dependencies

`../arch-docs-conventions/assets/template.md` · `../arch-docs-conventions/references/intent-contract.md` (id lifecycle, anchor drift, conflict artifacts)
