---
name: arch-spec-review
description: Use when a spec has landed (code is written) and, before merge, you must check the change against the recorded architecture intent for drift across marked boundaries. Precondition — every consumed architecture document must be ⏳-free, else halt for arch-why-elicit first.
---

# arch-spec-review · Architecture-level review of a landed spec

## Overview

A spec has landed; the code is written. Before it merges, check it against the **recorded architecture intent** (the documents built by `arch-doc-build` and completed by `arch-why-elicit`) and surface any **architecture-level drift** — places where the code now contradicts, crosses, or erodes a documented design intent, especially the marked boundary invariants (`🧱/💡/🚧`) and the section-level design rationales.

**Core principle**: This is a **read-only compliance check**. You hold the code to the recorded intent and **report** what you find — you invent no intent, make no architecture decision, **change no code, and change no document**. Only a **marked** item (`🧱/💡/🚧`) is enforceable; an `⏳` item is not yet an intent. A crossing of a marked boundary becomes a conflict artifact in the report; the legitimacy of the feature is orthogonal to whether it crossed a boundary — surface the crossing either way. **Deciding what to do about a finding — and any document change — belongs to `arch-doc-reconcile`, not here.**

**Violating the letter is violating the spirit**: a 🧱 waved through because "the need is reasonable" or "the tests are green" is still drift. Equally, "fixing" anything yourself — code or document — exceeds a compliance check.

**Run on the most capable model**: architecture-level drift detection is subtle, high-stakes judgment. This skill must be executed by the **best available subagent / model**, never a cheap one.

## When to use

- After a spec lands (code written), before merge, to review the change against recorded architecture intent.
- Invoked **standalone** (just a diff + the documents) or **by `arch-spec-flow`**, which additionally supplies the landed **spec document**. When a spec is supplied, its declared boundary-crossing intents (doc-path + INV-id) distinguish an **intended** architecture change from an **accidental** violation. You never go looking for the spec yourself — the orchestrator passes it (different flows keep specs at different paths).

**Do not use for**: building documents (`arch-doc-build`), updating WHAT after a change (`arch-doc-update`), proposing or carrying out document changes from a review's findings (`arch-doc-reconcile`), eliciting WHY (`arch-why-elicit`), or reviewing the **externally published** API (that is a product/spec responsibility, not architectural drift — see `intent-contract` §6).

## Iron Law

```
Review only on an ⏳-free map — any consumed document still bearing ⏳ → HALT for arch-why-elicit, do not review.
Only a marked item (🧱/💡/🚧) is enforceable; in the ⏳ region there is no violation judgment.
An undeclared crossing of a marked boundary → never auto-fix, never wave through "because the need is reasonable / the tests pass" → emit a conflict artifact; the merge waits behind the report.
A test that used to guard an invariant, flipped to assert the violation (or deleted), is drift — not a green light.
If a spec is provided: a crossing it declares (by doc-path + INV-id) carries the human's explicit authorization (granted at spec approval) — do not flag it as a violation, but verify the landed code matches the declared shape and scope. "Declared A, shipped B" (a different shape, or a wider scope than declared) → violation. A crossing the declaration did not name → violation. No spec → every crossing is a candidate violation.
You hold code to intent; you change nothing and decide nothing. Output is a compliance report (conflict artifacts + reconciliation signals) handed to arch-doc-reconcile / the human.
```

## Workflow

1. **Completeness gate (precondition)**. Determine the **consumed** documents: the directly matched ones plus every document whose invariants/content you actually use after expanding along the component graph. Run the gate's three machine checks (`intent-contract` "Completeness gate") on each. **Any `⏳` slot in a consumed document → HALT**: return "run `arch-why-elicit` to complete it, then resubmit". Do not review a partial map. (Documents merely passed over by a pointer, not consumed, do not trigger this.)
2. **Footprint + blast radius** (per `intent-contract` §6). For each changed file: attribute it to its **owning component** (coarse directory attribution, no mapping table); traverse the document graph's component edges (§2 seams, §5 drill-downs, preamble parent/child) for candidate radius; **calibrate by marked-boundary crossing** — if the change is component-internal and crosses no `🧱/💡/🚧`, radius is **local**; once it touches a marked boundary or a face another component's seam references, radius **expands** to all referencing parties. **Free finding**: a changed file whose owning component has **no document at all** = unregistered component = drift.
3. **Cross-boundary detection** (the core pass). For each changed region, hold it against the in-scope **marked** invariants and the section-level `> Rationale` of the owning and referencing components. A crossing = the code now contradicts or erodes a marked invariant or rationale. Check the rationale's **grounding pointer** too: if it reads "no registry import" and the code now imports the registry, the grounding is now false — that is the crossing made concrete.
4. **Guard-test inversion check**. Scan the **test** changes in the diff: did any test that asserted an invariant's behavior get **flipped to assert the opposite**, weakened, or deleted? A flipped/removed guard test is itself a finding and a strong signal the violation was made to pass CI — tie it to the invariant it used to guard. Green tests on an inverted guard validate nothing.
5. **Reconcile against the spec declaration (only if a spec was provided)**. A crossing the spec **explicitly declares it will make** carries the human's explicit authorization, granted at spec approval — review **trusts the declaration** (the same way the completeness gate trusts the document) and does **not** re-gate the decision. Its only job here is to confirm **intent ↔ landing consistency**. Match each crossing from steps 3–4 by **doc-path + INV-id**:
   - **declared, and the landed code realizes the declared shape — and only that** → **authorized**; not a violation → record as a **reconciliation signal** (the documents will need updating from the landed code, but the change is compliant), even for a 🧱.
   - **declared, but the landed code does something else** — a different shape than declared ("declared A, shipped B"), or a **wider scope** than declared (e.g. declared "relax on the fast path" but shipped "relax unconditionally for all runs") → **violation**: the authorization was for the declared shape, not this one.
   - **a crossing the declaration did not name** — including sibling invariants a declared change tramples but the declaration never listed (declare one secret 🧱, cross three), and emergent crossings the author may not have noticed (a flipped guard test, a widened contract) → **undeclared → violation** (not authorized this time).
   - **No spec provided** → skip this step; every marked-boundary crossing is a candidate violation.
6. **Triage** each remaining finding on three axes: **cross-intent boundary** (which marker) × **blast radius** (local / expands) × **intent gap** (is it in an `⏳` region — if so, no judgment). Classify the **root cause** per the conflict-artifact shape: design conflict / architecture / the rationale itself is wrong / **anchor drift** / code drifted away from intent.
7. **Classify each finding into the compliance report** — review **takes no action** on any of them (no code edit, no document edit, no routing):
   - **Authorized change** — declared and consistent (step 5: landed shape = declared shape, in scope), even for a 🧱; or a change that legitimately introduces a **new** architectural fact (not a violation). → record a **reconciliation signal**: compliant, but the documents will need reconciling from the landed code. The signal is for `arch-doc-reconcile`; review neither performs nor routes the update.
   - **Violation** — touches 🧱 / large radius / **declared-but-inconsistent** / **undeclared** / a flipped or removed guard test → emit a **conflict artifact** (shape below). Compliance fails; the merge waits behind it.
   - **`⏳` region** — no judgment (the gate already forced consumed docs complete, so a live `⏳` should appear only in a non-consumed corner).
8. **Severity flags** (still only reporting): flag a **large** deviation across the change as a `spec-design-failure` candidate (a signal that the spec may need redesigning), and the **same** architectural conflict recurring across reviews as an `architectural-problem` candidate (a signal that a 🧱 may need changing). Review only raises the flag in the report; whether to redesign the spec or move a 🧱 is decided **downstream** by the human / `arch-doc-reconcile`, never by review.

The entire process is agent-enforced and **read-only**; the only hard pre-checks are any invariants carrying `enforced_by` (`intent-contract` §8) — run those first, then make the soft judgment.

## Key judgments

### A reasonable need does not license crossing a 🧱

The most dangerous change is a sensible feature whose "obvious" implementation tramples a boundary — every step looks justified, the tests are green, the diff reads clean. Whether the need is legitimate is **orthogonal** to whether a boundary was crossed. Always surface the crossing; let the **human** reconcile need against intent (change the code, or consciously revise the invariant). Do not pre-absolve the change because the motivation is good.

### A spec declaration is the oracle for intended-vs-accidental

A landed diff cannot, by itself, tell a deliberate architecture change from an accidental violation — they look identical. The distinguishing signal is **design-time intent**, which lives in the **spec** (declared by doc-path + INV-id; see `arch-spec-flow`), not in the code. A declared crossing carries the human's **explicit authorization** (granted at spec approval); review trusts it and does not re-gate the decision — but authorization is for the **declared shape and scope**, not a blank check. So when a spec is provided:
- a crossing the spec **declared** is authorized — your question shifts from "is this a violation?" to "**did the code do exactly what was declared, and only that?**" Consistent → reconcile (even a 🧱). "Declared A, shipped B", or declared-narrow-shipped-wide → halt (you weren't authorized for that).
- a crossing the spec **did not declare** is a violation, full stop — including sibling invariants a declared change tramples but the declaration never named, and emergent ones the author may not have noticed (a flipped guard test, a widened contract).

Without a spec you cannot make this distinction: treat every crossing as a candidate violation and let the human supply the intent. Never infer "they probably meant to" from the code alone.

### Guard-test inversion is a first-class signal

A test is often the last line guarding an invariant. When a diff **rewrites that test to assert the violation**, two things happened at once: the invariant was crossed, and the safety net was cut so CI stays green.

<Good>
A test `"complete but verdict fails → continue (not settled)"` was rewritten to `"complete + verdict.ok=false → settle succeeded"`. Report it: it guarded `INV-…-complete-contract`; flipping it is part of the same drift, and the now-green suite is not evidence the behavior is correct.
</Good>

<Bad>
"The settle test passes, so the new settle behavior is fine." — the test was inverted to make it pass; it proves nothing.
</Bad>

### Only marked items are enforceable; the ⏳ region is silent

`🧱/💡/🚧` are intents you hold the code to. `⏳` is "intent not yet declared" — **no violation judgment, no subtraction** there (the gate already forced consumed docs complete, so a live `⏳` should only appear in a non-consumed corner). Do not invent a violation against an undeclared intent.

### Root-cause discriminator

Record the candidate root cause in the artifact; the column below is the **downstream** disposition that `arch-doc-reconcile` / the human will weigh — review only names the cause, it does not carry the disposition out.

| Root cause | Tell | Candidate downstream disposition (review records, does not act) |
|---|---|---|
| **Design conflict** | code **intentionally** does the opposite of a still-valid intent (the spec asked for it) | human reconciles need vs 🧱; if accepted, the invariant + rationale must be revised in the same change |
| **Code drift** | code **accidentally** violates a still-valid intent | candidate disposition = fix the code |
| **Anchor drift** | the intent's core still holds; only the WHAT symbol/structure it names moved with the code (`runTask`→`runEpisode`) | not a violation — a reconciliation signal: re-anchor via `arch-doc-reconcile`; human confirms |
| **Rationale wrong** | the recorded rationale never held up | human reconsiders the intent |

When the code deliberately does the opposite of a still-valid 🧱, the root cause is **design conflict**, not anchor drift — the symbol is unchanged, the rule changed.

### The gate trusts the document

The completeness gate is three mechanical checks (marker non-`⏳` + non-empty why + ≥1 grounding pointer). It does **not** re-derive the WHY or re-verify the rationale semantically — it trusts that the document went through the grounded creation flow. Your semantic work is holding the **code** to the marked intent, not re-auditing the document.

## Output (compliance report)

Review's whole output is a **report**, never an edit. It contains two kinds of entry:
- **Conflict artifacts** for every violation, in the shape shared with `arch-why-elicit` (`intent-contract` "Upgrade / conflict artifact") — plus any `spec-design-failure` / `architectural-problem` severity flags;
- **Reconciliation signals** for every compliant-but-document-affecting change (authorized declared crossings, genuinely new architectural surface, anchor drift): a short note of which document and which component the landed code now diverges from, so `arch-doc-reconcile` can propose the update.

A conflict artifact:

```
{
  violated/conflicting invariant id : INV-exec-core-complete-contract
  triggering signal                 : settle.ts success predicate dropped `verdict.ok`; guard test inverted
  intent (doc claims)               : 🧱 success = result.json:complete && verdict.ok (declaration + objective hard-verify); marked "immutable design decision"
  reality (actual code)             : success = complete only; verdict.ok no longer gates
  candidate root cause              : design conflict (spec deliberately relaxed a still-valid intent)
  options                           : revert relaxation / formally demote+rewrite the 🧱 in the same change
  should the document be updated    : only if the human accepts the relaxation
}
```

## Rationalizations

| Rationalization | Reality |
|---|---|
| "The product need is legitimate, so the change is fine" | Legitimacy of the need is orthogonal to boundary-crossing. Surface the crossing; the human reconciles. |
| "The tests are green / the guard test was updated to match" | A guard test flipped to assert the violation is drift, not validation. Green CI on an inverted guard proves nothing. |
| "It's just a small optimization, save the one hop" | A 🧱 is not tradeable against a micro-optimization. Emit the artifact. |
| "I can just fix this myself with a minimal diff" | Review is read-only. It edits nothing — not a 🧱 violation, not a reversible nit, not a document. Report the finding; `arch-doc-reconcile` turns it into a suggestion and a human adopts it. |
| "I'll route this to `arch-doc-update` / decide the doc should be rebuilt" | Deciding update-vs-build is `arch-doc-reconcile`'s job, and only as a suggestion a human adopts; the writing is `arch-doc-update` / `arch-doc-build`. Review only emits a reconciliation signal naming the divergence. |
| "The intent doc is probably outdated, the code is newer" | Maybe — but that is the human's call. Emit the artifact with root cause 'design conflict / doc may update'; do not silently side with the code. |
| "It's declared in the spec, so whatever shipped under that item is fine" | Authorization is for the **declared shape and scope**, not a blank check. "Declared A, shipped B" — or declared narrow, shipped wide, or a declared change that also trampled sibling invariants it never named — is not authorized; flag it as a violation. |
| "This `⏳` slot looks violated" | `⏳` is not an intent yet — no violation judgment there. |
| "I'll just list severities, that's clear enough" | Violations must be full conflict artifacts so `arch-doc-reconcile` / a spec redesign can consume them uniformly. |

## Red flags — stop

- Reviewing while a **consumed** document still bears `⏳` (gate not run).
- Editing anything at all — code **or a document**. Review is read-only; it never fixes a violation, a reversible nit, or a stale document.
- Routing or deciding the document reconciliation yourself (update-vs-build, "the doc should change") — that belongs to `arch-doc-reconcile`. Review emits only a reconciliation signal.
- Waving a crossing through because the feature is reasonable or the tests pass.
- Not scanning the test diff for a flipped/removed guard test.
- Emitting free-form severity prose instead of conflict artifacts for violations.
- Making the architecture decision yourself (revising a 🧱, blessing a relaxation) instead of reporting it.

**Any of these = you are inventing intent, making a decision that is not yours, editing outside a read-only check, or missing the safety-net cut. Back out: run the gate, report the violation as an artifact, change nothing.**

## Self-check

- [ ] Gate ran; every **consumed** document is `⏳`-free (else halted, did not review)
- [ ] Every changed file mapped to an owning component + blast radius; unregistered components flagged as free findings
- [ ] Every in-radius **marked** invariant and section rationale held against the code (grounding pointers re-checked)
- [ ] Test diff scanned; each flipped/removed guard test reported and tied to its invariant
- [ ] If a spec was provided: each crossing classified — declared+consistent (authorized → reconciliation signal, even a 🧱) / declared-but-inconsistent ("A vs B" or wider scope → violation) / undeclared (→ violation); authorized crossings not falsely flagged, inconsistent ones not waved through
- [ ] Each violation emitted as a full conflict artifact with a candidate root cause; each compliant-but-document-affecting change emitted as a reconciliation signal
- [ ] No judgment passed in the `⏳` region; no architecture decision taken; no reconciliation routed or performed
- [ ] Nothing modified — review is read-only (no code edit, no document edit)

## Dependencies

`../arch-docs-conventions/references/intent-contract.md` (Completeness gate · §3 markers/invariants · §5 seam/drill-down · §6 footprint/blast-radius · §8 `enforced_by` · the conflict-artifact shape) · the target architecture documents · the landed change's diff · the corresponding code · **optionally** the landed spec document (supplied by `arch-spec-flow`; when present its architecture-impact declaration drives the intended-vs-accidental classification). **Next phase on the report**: `arch-doc-reconcile` (read-only) consumes the conflict artifacts and reconciliation signals and turns them into **suggested** document changes; a human adopts the suggestions and `arch-doc-update` / `arch-doc-build` carry them out — or a spec redesign carries the artifacts. Review itself routes nothing.
