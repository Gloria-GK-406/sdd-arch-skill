---
name: arch-why-elicit
description: Use when an architecture document has its WHAT filled in but its WHY/strength markers are still ⏳ and a human is needed to complete them (after build/update produces output, before review's strict completeness gate requires ⏳-free). Interactive, human-in-the-loop; not dispatched by orchestrate.
---

# arch-why-elicit · Rationale elicitation and code grounding

## Overview

Given a document whose WHAT is filled in but whose WHY is entirely `⏳`, elicit the rationale for each decision from the human in interview style, cross-check it against the code via bidirectional grounding, then write it into the document and upgrade each `⏳` to a real strength marker.

**Core principle**: You only ask, only record, and only verify; you never invent. The sole legitimate source of a rationale is the designer's own words when questioned. A `⏳` with no human source stays. You are an interviewer, a recorder, and a fact-checker, not an author. This is the only skill in the set that may clear `⏳`.

Violating the letter is violating the spirit: passing off a code comment, a commit, or your own inference as a human answer is inventing.

## When to use

- After `arch-doc-build` / `arch-doc-update` produces a document containing `⏳`, to have a human complete the WHY.
- After `arch-spec-review`'s completeness gate halts on `⏳`, to complete it before re-reviewing.
- **Handling anchor drift from update**: when you scan a `⏳` slot that carries a `<!-- anchor drift -->` comment and still retains its old prose, the question is not "why?" from scratch but "can this old rationale migrate to the new structure?" — present the old prose plus the new code to the human: if it still holds, migrate it and restore the strength marker; if it does not, convert it to a conflict artifact.

**Do not use for**: building documents (`arch-doc-build`), changing WHAT (`arch-doc-update`), reviewing spec (`arch-spec-review`). **Not dispatched by `arch-doc-orchestrate`** — it must interview a real human, which a subagent cannot do; it runs only in the foreground, human-in-the-loop.

## Iron Law

```
No human source, the ⏳ stays. Never invent a rationale; never pass off a comment/inference as a human answer.
Grounding fails / there is a conflict → leave ⏳ or produce a conflict artifact; never force-clear.
```

## Workflow

1. **Scan out all `⏳` slots**: two kinds — item-level (the strength-marker slot of each atomic boundary) and block-level (the block-header strength of each `> Why built this way`).
2. **Generate a code-anchored question per slot** and ask the human (see "How to ask" below), focusing on one decision at a time.
3. **On receiving the human's answer, cross-check via bidirectional grounding** (see the two directions below).
4. **Choose one of three outcomes**:
   - **Grounded**: write the why block (the human's prose rationale), mark the slot's strength (`⏳`→`🧱/💡/🚧`), and attach a `→ code: path:line` grounding pointer → `⏳` cleared.
   - **Conflict** (answer contradicts the code): **do not clear `⏳`**; produce a conflict artifact in the shape defined by `intent-contract`, for the human to decide.
   - **No source** (the human cannot answer / the material does not cover it): **leave `⏳`**, record "the question to ask the human," and never fabricate.
5. **Judge partial grounding per slot**: one why block's prose may be shared by multiple slots, but whether to clear is judged per slot; **a whole section counts as complete if and only if every slot under it (block-level and item-level) is non-`⏳`**.

## Key judgments

### How to ask: anchor on the code, not in the abstract

<Good>
"`run.ts:114`'s `runEpisode` returns only `{kind:"aborted"}` on abort and does not judge the abort reason — why is the judgment of 'why it aborted' pushed to the coordination layer?"
</Good>

<Bad>
"Why is the execution core designed this way?" — the human has no starting point to answer, and any answer cannot be grounded.
</Bad>

### Bidirectional grounding: both directions must pass

- **Code→rationale**: can the rationale the human gave be corroborated from the current code? (Find the `path:line` that supports it.)
- **Rationale→code**: does the code's actual behavior, in reverse, confirm this rationale? **If they do not match, it is a conflict**; do not force grounding.

### Partial rationale: split per slot; do not wave it through as "broadly holds"

When a human rationale **partly still holds and partly has drifted**: use the part that holds to ground the corresponding slot (still subject to the three checks for clearing `⏳`), and convert the drifted part to a conflict artifact. Splitting is allowed, but it must be **judged per slot**; do not clear an entire rationale vaguely on the grounds that it "broadly still holds."

### Distinguish the root cause of a conflict (especially anchor drift)

| Root cause | Meaning | Handling |
|---|---|---|
| The rationale itself is wrong | The original rationale never held up | Produce an artifact; have the human reconsider |
| **Anchor drift** | **The rationale's core still holds; only the WHAT symbol/structure it depends on changed with the code** (the rationale refers to `runTask`, the code is now `runEpisode`) | Produce an artifact; the rationale may migrate, but the **WHAT anchor must be updated to match the code, and the human must confirm the migration; you do not ground it automatically** |
| Code drifted away from intent | The code now violates the original intent | Produce an artifact; a candidate root cause is that the code should change |

### Clearing `⏳` = three checks complete

Strength ∈ {🧱,💡,🚧} **and** the why block contains that rationale prose **and** at least one grounding pointer is attached. Missing any one, it does not clear.

### Only the settled rationale goes in the document

The why block receives **only the final rationale**, written as if it had always been there. **Never write the elicitation process into the document**: no "after discussing with the user…", no ghost / false answers that were detected and discarded, no "§x was corrected / judged real," no session audit trail. Such notes are architecture-irrelevant and pollute the document; they belong in the return report (see Output). The document shows the conclusion, not how it was reached.

## Rationalizations

| Rationalization | Reality |
|---|---|
| "The code comment states the reason, use it as the answer" | A comment is not the designer's words under questioning and may itself be a product of drift. Use it only as a question anchor, not as an answer. |
| "This rationale is too obvious, I answered for the human" | Answering for them is inventing and pollutes the true source of WHY. If no one answers, leave `⏳`. |
| "The core of the rationale holds, only the structure changed, I'll ground it onto the new structure directly" | That is anchor drift; the human must confirm the migration; do not ground automatically. |
| "This half still holds, count the whole as grounded" | Judge per slot. The drifted half must become a conflict; do not wave it through as "broadly holds." |
| "The human can't answer right now, I'll infer a plausible one and fill it in" | If they cannot answer, leave `⏳` — this is exactly the laziness/fabrication to guard against. |
| "I'll note in the document which answers were ghosts / that §x was corrected, for traceability" | That is process meta, irrelevant to the architecture, and it pollutes the document. Put it in the return report; the document shows only the settled rationale. |

## Output (return report)

Return these to the human / orchestrator — **never into the document**:
- **cleared `⏳`**: which slots, with the grounded rationale and pointer;
- **retained `⏳`**: the human could not answer — each with its code-anchored question;
- **conflict artifacts**;
- **session notes**: ghost / false answers detected and discarded, corrections made during the interview, anything about how the answers were obtained.

The document itself shows only the settled WHY.

## Red flags — stop

- A rationale the human never stated appears in a why block
- A code comment / commit is treated as a human answer
- `⏳` was cleared even though grounding did not pass
- A drifted half-rationale was grounded via "broadly holds"
- A rationale with anchor drift was grounded onto the new structure **automatically** (without the human confirming the migration)
- The document contains process / session notes (ghost-answer audit, "after discussing with the user", "identified and discarded", "judged real", which sections were corrected)

**Any of these = you are inventing, forcing grounding, or polluting the document with process. Back out: leave `⏳`, produce a conflict artifact, or move the note to the return report.**

## Self-check

- [ ] Every **cleared** `⏳`: strength marked + why prose comes from the human + grounding pointer attached (all three)
- [ ] Every **retained** `⏳`: accompanied by "the code-anchored question to ask the human," with no fabricated rationale
- [ ] All conflicts produced an artifact and did not clear `⏳`; anchor-drift cases were not migrated automatically
- [ ] Partial rationales split per slot, with no whole rationale grounded via "broadly holds"
- [ ] The document contains only settled rationale — no elicitation-process / session / ghost-answer notes (those went to the return report)
- [ ] No business code was modified

## Dependencies

`../arch-docs-conventions/references/intent-contract.md` (especially the "conflict artifact" shape and the three checks of the "completeness gate") · target document · corresponding code
