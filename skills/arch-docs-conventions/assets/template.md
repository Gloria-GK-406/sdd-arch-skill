# Architecture Design Document Template

<!-- ════════════════════════════════════════════════════════════════════
  A "fill-in" architecture design document template. Any system unit—whether
  the whole-service "overview" or a single component's "dedicated document"—is
  written with the same skeleton, so they are isomorphic: a component in the
  overview, drilled down, becomes a lower-level document of exactly the same
  shape.

  Usage:
    1. Copy this file; delete this note and all <!-- filling instruction --> comments.
    2. Mandatory 4 blocks: preamble / §1 structure overview / §3..N per-subpart detail / final drill-down.
    3. Optional 2 blocks: §2 assembly entry / end-to-end sequence—include per unit granularity; delete if absent.
    4. Fill each section's body per the "skeleton"; when unsure of the texture, see the <details> shape example at the end of the section.
       The example only demonstrates "what it looks like when filled in"; the A/B/methodA inside are placeholders—reuse the structure,
       replace with your real content.
    5. Rigid conventions (do not diverge):
       · Heading format `## N. <subpart name>`
       · Body opens with `**What**:`, stating structural facts only, no why mixed in
       · Tables must contain three column kinds: name / what it does / boundary
       · Code pointers fixed as `→ code: path:line`
       · why always goes in a blockquote, format `> **Rationale** <strength marker>`, and must carry a strength marker
  ════════════════════════════════════════════════════════════════════ -->

# <unit name> · <one-line positioning>

> <!-- Preamble: fixed three blocks, order unchanged. Each block 1–3 sentences, do not write long. -->
>
> **① Viewpoint level**: From which layer does this document look? Who is this
> unit, where in the system does it sit, who is above it (assembled/called by
> whom), who is below it (assembles/calls whom). Let the reader instantly
> locate which cell of the map they are in.
>
> **② Signal strength**: Each design tradeoff in the text is marked by strength—
> 🧱 **structural convention** (harder, change with care) /
> 💡 **recommended** (has a tradeoff basis, discussable) /
> 🚧 **tentative** (fastest-to-ship option, not deeply considered, likely to change).
>
> **③ Scope**: Is this document **guidance/recommendation** or **specification**?
> What it covers, what it does not. Fixed closing reminder: ⚠️ To really see how
> it runs, reading the code beats reading this document—this document only gives
> the skeleton and design rationale; behavioral details defer to `<code root path>`.

<!-- Filling instructions (delete in final):
  · All three blocks required, fixed order ①②③.
  · ① decides what mental model the reader carries onward; be sure to state the above/below relationships clearly.
  · ② the three symbol definitions are copied verbatim, shared across the whole document/project, do not invent per-document variants.
  · ③ the "guidance vs specification" must take a stance: which are hard (structural boundaries), which are debatable (internal implementation).
-->

---

## 1. Structure overview

<!-- Mandatory. First build a mental model with a diagram, then use a table to ground the diagram into "subpart → file → responsibility → boundary". -->

<This unit splits "<some task>" into <N> subparts with non-overlapping responsibilities (add a sentence on whether there is a hub spanning the whole flow, as needed):>

```
<ASCII structure diagram. Convention style:
  · A box ┌─┐ holds one subpart; inside the box write "name + one-line responsibility"
  · Arrows │▼ show data/control flow; label "what is passed" beside the arrow
  · A hub/shared state that spans the whole flow is drawn as a separate box, noting its read/write relationship with each segment>
```

| Subpart | File | One-line responsibility | Boundary (what it does not touch) |
|---|---|---|---|
| `<subpart A>` | `<path/a>` | <what it does> | <what it does not touch, what it does not know> |
| `<subpart B>` | `<path/b>` | <what it does> | <what it does not touch, what it does not know> |

> **Why split this way** <🧱/💡/🚧>
>
> <What is the split basis (e.g. rate of change, single responsibility, testable/replaceable)?
> Explain item by item why each subpart is independent and why the boundary is drawn this way.>

<!-- Filling instructions (delete in final):
  · The diagram + table + why trio is all required. The table is the precise form of the diagram.
  · The "boundary" column is key to isomorphism—it forces you to think clearly about what each subpart "does not do".
  · The split's why carries a strength marker; a structural-level split is usually 🧱.
-->

<details>
<summary>📖 Shape example (placeholder content, reuse the structure)</summary>

This unit splits "<one processing run>" into 3 subparts with non-overlapping responsibilities, plus a hub H spanning the whole flow:

```
┌─────────────────────────────────────────┐
│  subpart A   —— <one-line responsibility> │
└─────────────────────────────────────────┘
            │ <what is passed A→B>
            ▼
┌─────────────────────────────────────────┐
│  subpart B   —— <one-line responsibility> │
└─────────────────────────────────────────┘
            │ <what is passed B→C>
            ▼
┌─────────────────────────────────────────┐
│  subpart C   —— <one-line responsibility> │
└─────────────────────────────────────────┘

   ┌────────────────────────────────────┐
   │  hub H (spans the whole flow)       │
   │  writes: A/C · reads: B             │
   └────────────────────────────────────┘
```

| Subpart | File | One-line responsibility | Boundary |
|---|---|---|---|
| A | `path/a` | <what it does> | does not touch <X>; does not know <Y> |
| B | `path/b` | <what it does> | does not touch <X>; does not know <Y> |
| C | `path/c` | <what it does> | does not touch <X>; does not know <Y> |
| H | `path/h` | bridges A/C's writes and B's reads | pure in-memory; transient, not persisted |

> **Why split this way** 🧱 (layering) / 💡 (<characteristic of some subpart>)
>
> The split basis is <e.g. rate of change>: isolate <what changes> from <what should stay stable>. A only carries <…>, B is <…>, and C is thereby able to <stay stable / be unit-testable / be replaceable>.

</details>

---

## 2. Assembly entry / seam

<!-- Optional. The overview usually has it (describing how the composition root news up each implementation);
     components often omit it, or it degenerates into "who injects me, what is injected, where is the seam".
     Delete the whole section if absent. -->

<How this unit is assembled / at which composition root it is newed / through which seam it plugs into the layer above.>

```
<assembly tree: composition root ── new implementation A (seam #1)
                ├─ new implementation B (seam #2)
                └─ ... ──▶ injection container ──▶ startup>
```

→ code: `<path/entry>`

> **Rationale** <💡/🚧>
>
> <The assembly approach's tradeoff: why assembled this way, which dependencies are optional and why.>

<details>
<summary>📖 Shape example (placeholder content)</summary>

`<entry file>` is the sole **composition root**: it news up each concrete implementation, packs them into the dependency container, and hands off to `<build()>` to connect to the layer above. `<implementation A/B/hub>` are required; `<implementation C/D>` are optional (default semantics = <…>).

> **Rationale** 💡 (assembly) / 🚧 (optional dependencies)
>
> Use a "<unified assembly point / some approach>" rather than <some alternative>—because <reason>. Optional dependencies are the current fastest option, expected to change in the future.

</details>

---

## 3. <subpart name>

<!-- Mandatory, and every subpart worth its own section follows this same shape:
     What → table/diagram → code pointer → why blockquote.
     §3, §4, §5… repeat this section per the number of subparts. -->

**What**: <one-line structural fact—what this subpart is, what role it carries, what shape it follows>.

<Choose the table that fits best: routing table / interface table / method table / state field table. The header rigidly contains three column kinds:>

| Name · entry | What it does | Key convention / boundary |
|---|---|---|
| `<methodA()>` | <what it does> | <what it returns / what precondition / what it does not do> |
| `<methodB()>` | <what it does> | <…> |

→ code: `<path:line>`

> **Rationale** <💡/🧱/🚧>
>
> <Why chosen this way—current thinking, not an iron law. May point out compromises, costs, and where it is open to revisiting.>

<!-- Filling instructions (delete in final):
  · "What" states structural facts only, never mixing in why; all why goes in the blockquote.
  · The table gives at least the three kinds "name / what it does / boundary"; column names may change but the three kinds must be present.
  · Every subpart must have a why blockquote—a subpart for which no why can be written probably should not be its own
    section, and should be merged back into an adjacent section or demoted to a single line.
  · The blockquote must carry a strength marker. Compromise designs and tentative options should be honestly marked 🚧.
-->

<details>
<summary>📖 Shape example (placeholder content)</summary>

**What**: <what subpart A is>, and all <entries> follow the same shape—<some fixed flow>, with no business logic written in <some place>.

| Name · entry | What it does | Key convention / boundary |
|---|---|---|
| `entryA` | <what it does> | <precondition / return / what it does not do> |
| `entryB` | <what it does> | idempotent; unknown input is a no-op |
| `entryC` | <what it does> | <…> |

→ code: `path/a:NN-NN`

> **Rationale** 💡 (<tradeoff title, e.g.: compromise design>)
>
> <Why chosen this way>. The cost is <some duplication / some constraint>—this is a compromise we accept, not debt to be cleared.

</details>

---

## 4. <subpart name>

<!-- Repeat the shape of §3. Delete this hint; add or remove sections as needed. -->

**What**: <…>.

| Name · entry | What it does | Key convention / boundary |
|---|---|---|
| … | … | … |

→ code: `<path:line>`

> **Rationale** <strength marker>
>
> <…>

---

## N. End-to-end flow / sequence

<!-- Optional. "Strings together" the preceding subparts in one complete run. The overview usually has it; component level
     rarely needs it (a component is usually just one link in the flow). Delete the whole section if absent. -->

<String the subparts of §3..N together along one real flow, showing how they collaborate.>

```
<sequence diagram. Convention: participants labeled on the left, ──message──▶ marks a call, vertical axis is time,
  use indentation/branches for "background async", "concurrent subscription", etc.>
```

→ code: `<relevant entry path:line>`

> **Rationale** <strength marker>
>
> <Design points in this flow worth explaining—especially cross-subpart collaboration conventions.>

<details>
<summary>📖 Shape example (placeholder content)</summary>

```
external ──request──▶ A
               │ <precheck>
               │ delegate B ──────────┐ background
               └──acceptance reply──▶ external │
                                   ▼
                                 B orchestration
                                   create entry → call C
                                   C ──▶ hub H (write progress)
                        external ──subscribe──▶ A ──▶ read from H
                                   finalize terminal state → persist
                        external ──fetch result──▶ A returns terminal state
```

</details>

---

## Final. Collaborating components / drill-down

<!-- Mandatory. The "mount point" of isomorphic recursion: subunits outside this document's scope but depended on or
     called by this unit, each given only "one-line responsibility + seam + dedicated-document pointer", with details
     ceded to that dedicated document. Each pointer points to a lower-level document written with this same template,
     isomorphic to it. -->

Beyond the main axis/main body, the following subunits are injected or called, each with a dedicated document; this document only marks responsibility and seam:

| Component | Seam / role | One-line responsibility | Dedicated document |
|---|---|---|---|
| `<component A>` | <seam #N / what role> | <one line> | <link / `to-be-written` / `—`> |
| `<component B>` | <…> | <…> | <…> |

<!-- Filling instructions (delete in final):
  · This table is the "drill-down" index. Each row = one potential isomorphic lower-level document.
  · "Seam/role" states clearly through which interface it plugs into this unit (the seam number echoes the diagram in §1/§2).
  · The dedicated-document pointer has three states (see intent-contract §5):
      - a link — the dedicated document exists;
      - `to-be-written` — the component warrants its own document but is not written yet (decomposition signal; marks a document still missing in the tree);
      - `—` — a one-line row suffices; no dedicated document (a permanent leaf).
    Do not leave it blank.
-->
