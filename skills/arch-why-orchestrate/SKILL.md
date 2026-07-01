---
name: arch-why-orchestrate
description: Use when an architecture document tree has WHAT filled in but WHY still largely ⏳ and needs systematic completion of WHY (after arch-doc-orchestrate). Foreground, sequential, human-in-the-loop; does not fan-out subagents.
---

# arch-why-orchestrate · Rationale interview orchestration

## Overview

Across a document tree where **WHAT is filled but WHY is still largely `⏳`**, systematically complete WHY: scan for `⏳`, order them, run `arch-why-elicit` in the foreground one item at a time to interview the person, and **check across documents that the rationales do not contradict each other**.

**Core principle**: You are the **interview orchestrator** — you decide **what to ask, in what order, and whether the answers are mutually consistent**. The specifics of **how to ask a single item** (code-anchored questions plus two-way grounding) belong to `arch-why-elicit`. You do not ask specifics yourself, do not invent rationales, do not resolve conflicts on the person's behalf, and do not change WHAT.

**Asymmetry with `arch-doc-orchestrate`**: This skill **cannot fan-out subagents**. `arch-why-elicit` interviews a real person, which a subagent cannot do; there is only one person, and their answers must stay consistent. The interview is therefore **foreground, sequential, and human-in-the-loop**, not parallel work.

## When to use

- A document tree where WHAT is filled but WHY is largely `⏳` (after `arch-doc-orchestrate`), to systematically complete WHY.

**Do not use for**: a single WHY item or document (use `arch-why-elicit` directly), building or refreshing WHAT (`arch-doc-orchestrate`), or reviewing a spec (`arch-spec-review`).

## How to ask vs what to ask

| `arch-why-elicit` | This skill |
|---|---|
| **How to ask a single item**: code-anchored questions + two-way grounding + three outcomes (grounded / conflict / leave ⏳) | **What to ask + in what order + whether answers are consistent**: scan, order, close out cross-document consistency |

## Workflow

1. **Scan the whole tree for `⏳`**: list every `⏳` slot (item-level boundaries and block-level why-block headers), noting the owning document and dependency relationships.
2. **Order (foundation-first)**: see "Ordering strategy" below.
3. **Drive `arch-why-elicit` for each item**: in order, invoke it in the foreground for each `⏳` to interview the person and ground the answer. **Pass along already-settled upstream WHY** as question context so the person can reference it rather than restate it.
4. **Cross-document consistency check**: after each batch of answers, check that they do not contradict each other or already-settled WHY (see "Consistency check" below).
5. **Conflict → escalate to the person**: when rationales contradict each other, produce a "conflict artifact" for the person to resolve. Do not smooth it over yourself.
6. **Close-out report**: cleared `⏳` / remaining `⏳` (person could not answer) / conflicts / cross-document inconsistencies.

## Ordering strategy

**Topological order, foundation-first**:

- **Overview WHY before component documents** — the overview's partitioning and assembly decisions are the context for component decisions.
- **Within a document**: §1 structural decisions such as partitioning and assembly (🧱 candidates) first, details after; ask first whatever **other invariants depend on**.
- **Across documents**: hubs and shared components depended on by many parties first, leaves and consumers after.

Rationale for this order: once upstream is settled, downstream can **reference rather than restate**; contradictions surface early at the spine; the person follows a top-down narrative and tires less. Asking out of order (leaves before foundation) wastes effort and invites self-contradiction.

## Cross-document consistency check

`arch-why-elicit` catches drift between **intent and code**; this skill catches **intent vs. intent** contradictions across the tree. Check:

- whether a component's WHY violates the overview's partitioning WHY;
- whether two interacting components hold consistent assumptions about a **shared seam**;
- whether the rationales for similar 🧱 across documents conflict.

An inconsistency is a **real signal** — either a misunderstanding or a genuine architectural tension → escalate to the person to resolve; do not smooth it over yourself.

## What it does not do

- Does not ask specifics (that is `arch-why-elicit`), does not invent rationales, does not resolve conflicts on the person's behalf, does not change WHAT, does not assign weight, and does not fan-out.

## Durable progress

Like `arch-doc-orchestrate`, keep a **ledger** (which `⏳` are cleared / asked / deferred). Interviews often span multiple sessions — resume from the ledger and do not re-ask what is already answered.

## Output

- A document tree with `⏳` cleared as far as possible (each cleared item satisfies the three grounding requirements: weight + non-empty WHY + grounding pointer);
- A list of **remaining `⏳`** (person could not answer) + **conflict artifacts** + a list of **cross-document inconsistencies**.

## Red flags — stop

- **Fan-out subagents to "interview in parallel"** — there is only one person and answers must stay consistent; it must be sequential.
- **Inventing rationales** yourself or **smoothing over contradictions** yourself (both should go to the person).
- **Not ordering / asking out of order** (leaves before foundation wastes effort and invites contradiction).
- **Skipping the consistency check** (each document's WHY is internally coherent but they conflict together).
- Changing WHAT or assigning weight beyond your role (that is build/update work, or the person's).

## Dependencies / integration

- **Calls**: `arch-why-elicit` (per-item interview + grounding).
- **Upstream**: `arch-doc-orchestrate` (produces a tree with WHAT filled and WHY `⏳`).
- **Basis**: `../arch-docs-conventions/references/intent-contract.md` (`⏳`, conflict artifacts, completeness gate).
- **Downstream**: once `⏳` are cleared → the completeness gate in `arch-spec-review` can pass.
