# intent-contract · machine-readable conventions for architecture documents

> `arch-doc-build` / `arch-doc-update` **produce** documents per this contract; `arch-why-elicit` **clears ⏳** per this contract;
> `arch-spec-review` / `arch-doc-orchestrate` **parse** documents per this contract.
> This is the interface contract between the WHAT writer, the WHY filler, and the review/orchestration consumers.

---

## Core model (cross-section; read this first)

1. **Ownership split**: WHAT (structural facts) is written by the skill; WHY (rationale) is written and owned by a human; the `🧱/💡/🚧` strength marker is set by a human.
   - **The only valid source of a WHY is the rationale the designer gives when questioned.** Code comments, commit messages, and other "implementation-site utterances" must not be treated as the human source of a WHY — a comment may itself be a product of drift. At most it serves as a questioning anchor for `arch-why-elicit`; it must never be passed off as a human answer and written into a why block.
2. **Dual-state boundary cells**: a boundary is born as a descriptive fact plus pending (`⏳`, no strength marker, written by build); after human review it is assigned a strength marker and upgraded to a prescriptive invariant (`⏳` cleared). `⏳` and the strength marker occupy the same slot — `⏳` is the "strength marker pending human input" state.
3. **Zero scripts, agent-enforced (no scripts)**: every convention in this contract is enforced by the agent reading this file; there is no script-based hard check. The sole exception is §8 `enforced_by` — a per-item, opt-in deterministic hard safeguard.
   - Trade-off (for awareness): agent enforcement is non-deterministic; dangling pointers and misalignments are occasionally missed. In return, there is zero script infrastructure and everything stays in the skill. Most of what this contract handles is architectural judgment that cannot be reduced to a regex, so maintaining scripts for the few mechanical items is not worthwhile.
4. **Strict review gate**: see "Completeness gate" below.
5. **Closed loop**: `build/update produces ⏳ → human runs arch-why-elicit (questioning + bidirectional code grounding, clears ⏳) → review strict gate admits`.

---

## Document language

The architecture documents produced by these skills default to the language the user works in. The labels and prose in this contract and in `template.md` are the canonical English reference; when producing a document, render the section labels and prose in the target language. The functional tokens — the 🧱/💡/🚧/⏳ markers, `INV-` ids, the boundary-cell format, and code pointers — stay constant across all languages.

---

## Document content: present architecture only

A document records the architecture **as it is now** — nothing else. It must not record:
- **History** — superseded structure, "formerly X", retired ids or numbers (history lives in git; see also "no tombstones" in §4).
- **Process / provenance** — how the content was produced: elicitation-session notes, "after discussing with the user…", ghost / false answers that were detected and discarded, which sections were corrected, or any audit trail of the Q&A.

Only the settled, architecture-relevant result belongs in the document; a why block holds only the final rationale, written as if it had always been there. Process and session notes belong in a skill's **return report** to the human/orchestrator, never in the document.

This does not forbid the **functional machinery** — `⏳`, and the transient `<!-- anchor drift -->` handoff marker that update plants for why-elicit. Those are structured, transient state that gets resolved and removed, not free-form provenance prose. The rule targets narrative process/history notes, not the contract's tokens.

---

## Document location and naming

**Default location**: architecture documents live under `docs/architecture/`. In a monorepo / aggregate project, namespace by sub-project: `docs/architecture/<project-or-subsystem>/`.

**Tree layout**: each unit is a directory; its overview is `00-<name>.md`. Component documents below it are numbered `01-<name>.md`, `02-…`, in the order of the overview's drill-down table. A component that itself decomposes becomes a sub-directory (`03-<component>/00-<name>.md` plus its own children); a leaf component is a single file (`03-<component>.md`).

**The `NN-` prefix is reorderable reading order, not an identity.** Identity and all cross-document links resolve by the stable `<unit>` code; the number is only a human-facing ordering hint that mirrors the overview's drill-down order. update may renumber sibling files freely to keep the order coherent (links use `<unit>`, so renumbering is cosmetic). History lives in git, not in the numbers — do not keep gaps or "retired" numbers as placeholders (same principle as no tombstones).

```
docs/architecture/execution-plane/
├── 00-main-structure.md      (overview)
├── 01-run-registry.md        (leaf component)
├── 02-run-service.md
└── 03-runtime/               (decomposes → directory)
    ├── 00-runtime-overview.md
    └── 01-opencode-adapter.md
```

---

## Completeness gate (precondition for review)

**Review performs subtraction only on an intent-complete map.** Every document that review references and actually consumes (the directly matched documents plus all documents whose invariants/content are actually used after expanding along the component graph) **must be entirely `⏳`-free**; otherwise the whole run **halts** and the human is required to first run `arch-why-elicit` to complete it, then resubmit for review. This is the forcing function against shortcutting and against stale documents. (Documents merely passed over by a pointer, whose content is not consumed, do not trigger this.)

**The gate performs only three machine checks** (it trusts that the document went through the full grounded creation flow and does **not** re-check semantics):
1. Every in-scope invariant has a strength marker ∈ {🧱,💡,🚧} (not `⏳`);
2. The corresponding why block is **non-empty** (has prose rationale);
3. The why block carries **at least one** `→ code: path:line` grounding pointer.

Missing any of the three → halt.

**The true unit of completeness is "each slot", not "each why block".** A document has **two kinds of `⏳`-bearing slots**, cleared independently:
1. **Item-level**: the strength-marker slot of each atomic boundary invariant (inside the boundary cell, §3).
2. **Block-level**: the **block-header strength marker** of each `> **Rationale** <strength>` block — it marks the strength of that section's **overall design decision** (e.g. "why split this way", "why assembled this way"), which is a different object from a boundary constraint.

The **prose of one why block may be shared by multiple invariants**, but **grounding is judged per slot**: a slot is grounded ⟺ its own strength marker is non-`⏳` **and** the why block contains its rationale plus a grounding pointer. So when a block covers 3 items but explains only 2, the 3rd item's strength slot **stays `⏳`** and the gate still halts — prose may grow incrementally, and **a section is complete iff every slot under it (block-level plus each item-level) has a non-`⏳` strength marker**.

---

## 1. Document skeleton

Follow the rigid conventions of `template.md` (delete the template's filling guidance and shape examples from the finished document):
- Title format `## N. <subpart name>`
- Body begins with `**What**:` and **records only structural facts, with no why mixed in**
- Tables must contain three column types: name / function / boundary
- Code pointers use the fixed form `→ code: path:line`
- WHY goes into a blockquote, format `> **Rationale** <strength>`, **must carry a strength marker**
- The preamble declares this unit's **`<unit>` code** (see §4), in the format below

**Preamble `<unit>` declaration**: **on its own line, immediately after the preamble's ① perspective-layer block and before ② signal strength**:
```
> **Unit code**: `exec`
```
build automatically derives a slug from the unit name/path and writes it; a human may change it. It is this document's id namespace prefix and its identity when stitching the document tree.

## 2. Strength markers

- `🧱` structural convention (hard; change with caution)
- `💡` recommended (has rationale, discussable)
- `🚧` tentative (fastest path, likely to change)
- `⏳` **pending** (WHAT written; strength marker and WHY pending human input) — **same slot, mutually exclusive** with the three markers above

Strength (`🧱/💡/🚧`) and "whether a machine check exists" (§8 `enforced_by`) are **orthogonal**: absence of `enforced_by` does not mean weak; the hardest 🧱 is often precisely the pure architectural intent that code analysis cannot catch.

## 3. Boundary / invariant (review's "cross-intent boundary" axis)

**Atomic granularity**: in the boundary column, **each atomic statement = one invariant**, each carrying its own strength marker and id. When a cell holds multiple invariants, write them line by line (separated within the cell by `<br>`); mixed strengths are allowed.

**Inline format**: `<strength or ⏳>[<id>] <statement>`

Dual-state example (same cell, build output → after human upgrade):
```
build output:    ⏳[INV-exec-no-registry] does not touch the registry <br> ⏳[INV-exec-no-http] does not emit HTTP for now
after upgrade:   🧱[INV-exec-no-registry] does not touch the registry <br> 🚧[INV-exec-no-http] does not emit HTTP for now
```

- Review **enforces only items with an assigned strength marker**; an `⏳` item = not yet an intent = **no subtraction, no violation judgment** (but it is caught by the completeness gate).
- One id must point precisely to **one assertion that can be independently violated and independently up/downgraded**.

**Two criteria for minting an id** (avoid overproduction, avoid arbitrary splitting):
- **Qualify before entering the boundary column**: the boundary column holds **architecturally meaningful constraints** (what this component must not know/touch/do; hard cross-layer boundaries), **not every defensive coding practice or implementation detail**. When unsure whether an item is an implementation detail, write it as `⏳` and let the human decide or delete it at upgrade time — but do not pack the boundary column with low-value assertions and burden human review.
- **Restating sections mint no new id**: sections that "string the subparts together and restate the collaboration" (end-to-end flow, sequencing, etc.) **introduce no new boundary invariant and mint no new `INV-`** (if such a section's collaboration design point is worth explaining, it may still have a **block-level** why with a block-header strength marker, but it produces no item-level invariant).

## 4. Stable id

**Format**: `INV-<unit>-<slug>`
- Prefix `INV-`: a single namespace; **strength is not encoded in the prefix** (strength has one true source — the strength-marker slot — to avoid prefix/marker drift).
- `<unit>`: the unit code declared in this document's preamble (see §1); it achieves project-wide global uniqueness with no central registry.
- `<slug>`: a **semantic kebab** derived from the statement (`no-http`), not a sequence number (sequence numbers carry no meaning and break on insert/delete).

**Lifecycle**:
- build **mints once** at WHAT time (writes `⏳[INV-exec-no-http] …`).
- **Statement rewritten, id unchanged**: the same assertion's id is stable across updates.
- update **keeps existing ids**, minting new ids only for new boundaries.
- **Deletion is deletion**: if code removes a constraint, delete the corresponding boundary line together with its id — **no tombstone, no historical residue**. The document records only the present; history lives in git. Ids are semantic slugs, so a new, different invariant naturally gets a different slug (no collision); if a semantically identical invariant is later reconstructed, reusing the same name is in fact correct.

## 5. Seam / drill-down section (the edges of the document tree)

Drill-down table in the final section: `component | seam/role | one-line responsibility | dedicated-document pointer`.

**Two machine-load-bearing elements**:
1. **Dedicated-document pointer**, one of three states:
   - a **link** — the dedicated document exists;
   - `to-be-written` — the component **warrants** its own document but it is not written yet (the decomposition signal and orchestrate's to-do queue);
   - `—` — **a one-line row suffices; no dedicated document** (a permanent leaf, never drilled into).

   This is the **edge** of the document tree; orchestrate materializes every `to-be-written` into a document and recurses, and never builds a `—` row.

   **build decides the state** at decomposition time (the same family of judgment as main-axis-vs-drill-down): a component gets `to-be-written` when it is self-contained, has an independent rate of change, and is **complex enough that a one-line row cannot capture it / it has internal structure worth its own isomorphic document**; otherwise `—`. This bounds documentation sprawl the way the boundary-column "qualify before including" rule bounds invariant overproduction.

   **File vs directory** is decided when a child's own build runs: if the child's drill-down table has any `to-be-written` rows it is a branch → a directory (`<n>-<child>/00-<name>.md`); otherwise a leaf → a single file (`<n>-<child>.md`). Over-/under-decomposition self-corrects: update (or a human) collapses an over-split document back into a one-line `—` row, or adds a `to-be-written` later — the drill-down table is editable.
2. **`<unit>` alignment**: the child document a pointer points to must declare a `<unit>` in its preamble that matches the pointer.

**Drill-down rows mint no `INV-` id**: this document records only "component + seam + one-line responsibility + pointer"; that component's **invariants are minted in its own dedicated document** (using that document's own `<unit>`). This document does not mint ids on behalf of child components.

**Stitching-time checks (agent-enforced, no scripts) — two tree invariants**:
- Every drill-down pointer resolves to a real child document, and the child's `<unit>` matches the pointer (guards against dangling/mis-pointing);
- Each child document is **pointed to by exactly one parent** (no orphans, no two parents) — guaranteeing that the tree whose union equals the project architecture map is a real tree.

**Purely human-readable, no machine dependency**: seam numbers `#N` (echoing the table↔diagram cross-reference anchors of the §1/§2 ASCII diagram).

## 6. footprint and the "blast radius" axis

**No file-level mapping table.** Blast radius is a semantic/relational property living at **component granularity**, determined by the document graph plus LLM judgment:

1. Changed file → **owning component** (only a coarse directory/folder attribution is needed, not a mapping table to maintain).
2. **Traverse the document graph's component-reference edges** (§2 seams, §5 drill-downs, preamble parent/child) to find candidate radius.
3. **Calibrate using boundary markers**: if the change touches only component internals and no `🧱/💡/🚧` boundary (the contract is unchanged) → actual radius is **local**, even if widely referenced; once it touches a **marked boundary / the face referenced by another component's seam** → radius expands to all referencing parties.

**Free finding**: if the changed file's **owning component has no document at all** = unregistered component = a **drift finding**.

> The "public faces" within the system are covered by the traversal above; **the externally published API is out of scope for this skill** (that is a product/spec-layer responsibility, not architectural drift). Hence there is no publicness annotation.

## 7. (removed) public API surface

Intentionally removed. See the end of §6.

## 8. Executable hook `enforced_by` (optional, presence-triggered)

**If real static/dynamic analysis can catch it → attach; otherwise → the field is absent.** By default everything is an agent-enforced soft constraint; `enforced_by` is a per-item, opt-in deterministic hard safeguard, used only for the few critical invariants too costly to miss.

**Form** (placed in the invariant's why block, or immediately after the boundary line):
```
enforced_by: [import-linter#exec-core-no-registry]
```
When review/orchestrate encounters an item with `enforced_by`, it **runs that check first**, then makes the soft judgment.

**Two guardrails**:
1. **Never attach a placeholder**: once `enforced_by` appears, the referenced check must **really exist and be runnable**. Do not write `[TODO later]` — that would make review trust a safeguard that never ran. If there is none, leave it empty.
2. **Attachment timing**: it attaches to an already-formed 🧱 (`⏳` cleared, human has confirmed the intent), added by a human or `arch-why-elicit` only when it is known that "such a rule really guards it".

---

## Upgrade / conflict artifact (shared by rationale and review)

`arch-why-elicit` (when it finds a human-given rationale contradicts the code) and `arch-spec-review` (when it halts and hands off to a human) produce **the same-shaped** artifact:

```
{
  violated/conflicting invariant id : INV-exec-no-http
  triggering signal                 : <which rule / which code triggered>
  intent (doc/material claims)      : <statement from doc or human rationale + strength>
  reality (actual code)             : <what the code actually does>
  candidate root cause              : design / architecture / the rationale itself is wrong / anchor drift / code drifted away from intent
  options                           : <available dispositions>
  should the document be updated    : yes / no
}
```

**Root cause `anchor drift`** (a distinct category): **the rationale's core still holds; only the WHAT symbol/structure it attaches to has changed with the code** (e.g. the rationale speaks of `runTask` but the code is refactored to `runEpisode`; the rationale speaks of inline verify, now extracted into `checkTurn`). This is **neither "the rationale is wrong" nor "the intent was violated"** — the disposition is: the rationale can migrate, but the **WHAT anchor must be updated to track the code and a human must confirm the migration**; `arch-why-elicit` **does not auto-ground** it.

**Partial-rationale rule**: a human/material rationale that **partly still holds and partly has drifted** is **neither grounded wholesale nor discarded wholesale** — the holding part grounds the corresponding slot (still subject to the three grounding requirements each), and the drifted part becomes a **conflict artifact**. Partial use is allowed, but it must be judged **per slot** and must not be glossed over as "broadly holds".

**Cross-section same-symbol drift** (a separate axis): drift of one code symbol (e.g. `runTask`→`runEpisode`) may simultaneously hit **multiple sections' WHY blocks**. Each WHY block is an **independent slot**, handled independently (update demotes each to `⏳` + marks; why-elicit re-reviews each) — **do not mark/migrate only one place**. This is distinct from "splitting use within a single rationale" above.

- `arch-why-elicit` on conflict **does not clear ⏳**; it emits this artifact for the human to decide (change the code? or was the original intent wrong?).
- `arch-spec-review` with large deviation → judges the spec design a failure → **redesigns the spec with the artifact** (rather than blindly rerunning); the same architectural conflict recurring → judges it an architectural problem and escalates to changing the 🧱.
