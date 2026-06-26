# arch-skills

A group of **Agent Skills** for building **isomorphic architecture-intent documents** and doing **architecture-level review** against them — so an AI coding agent can recognize and stop architectural drift early, and so the same documents stay in sync inside a spec-driven development (SDD) flow.

**Goal**: not to keep the architecture entirely clean, but to preserve cleanliness at a given scale, **ensuring that the cost of later targeted refactoring does not grow too large** (i.e., preventing the "refactoring window" from closing).

Works across agents that read the Agent Skills format: **Claude Code, Codex, Cursor, Copilot CLI, Gemini CLI, Kimi**, and others.

## Install

> **⚠️ Install the whole group — never cherry-pick a single skill.** The skills share a home skill, `arch-docs-conventions` (it holds the shared `template.md` and `intent-contract.md`), and reference it with relative paths like `../arch-docs-conventions/references/intent-contract.md`. Installing one skill alone breaks those references. Always install all of them (siblings must sit together under the same skills directory).

### Any agent — via `npx skills` (recommended)

[`npx skills`](https://github.com/vercel-labs/skills) installs into the cross-runtime `~/.agents/skills/` directory (which Codex, Copilot CLI, and Gemini CLI read) and symlinks into each agent's own directory:

```bash
# install the whole group globally for Codex + Claude Code
npx skills add Gloria-GK-406/sdd-arch-skill -a codex -a claude-code -g --all

# other useful commands
npx skills list                    # what's installed
npx skills remove sdd-arch-skill   # uninstall
```

- Add `--copy` to copy files instead of symlinking (use this if your agent doesn't follow symlinks — a known case for Claude Code).
- `-a` selects agents (`codex`, `claude-code`, `cursor`, `opencode`, …); omit `-a` to be prompted.

### Claude Code — native plugin / marketplace

```text
/plugin marketplace add Gloria-GK-406/sdd-arch-skill
/plugin install arch-skills@arch-skills-dev
```

…or load it directly with `--plugin-dir <path-to-arch-skills>`, or drop the `skills/*` directories into `~/.claude/skills/`.

### Codex — manual

Copy the skill directories into Codex's skills directory:

```bash
cp -r skills/* ~/.codex/skills/      # or ~/.agents/skills/
```

Codex auto-discovers every subdirectory that contains a `SKILL.md`.

## Skills

| skill | role | concern |
|---|---|---|
| **`arch-docs-conventions`** | entry / shared home | holds the shared `template.md` + `intent-contract.md` (the writing/parsing conventions everything reads); the other skills reference it |
| **`arch-doc-build`** | build WHAT | whether a document exists — if absent, fill WHAT from scratch and leave every marker as `⏳` |
| **`arch-doc-update`** | update WHAT | whether the code changed — apply a **minimal patch**, **preserve human-written WHY**, mark new points `⏳` |
| **`arch-why-elicit`** | fill WHY | elicit rationale by **questioning** the human + ground it **bidirectionally** against code; the **only** skill that clears `⏳`; only asks, records, verifies — never invents |
| **`arch-spec-review`** | review | **read-only** compliance check: hold a landed change to the recorded intent and **report** drift across marked boundaries as conflict artifacts + reconciliation signals; changes nothing, decides nothing; gated by a completeness gate up front |
| **`arch-doc-reconcile`** | suggest doc changes | identify where the documents diverge from landed code and produce a **suggestion report** (which change + which atom fits); **read-only** — writes nothing, dispatches nothing; a human adopts the suggestions, then `arch-doc-update` / `arch-doc-build` carry them out |
| **`arch-doc-orchestrate`** | WHAT-tree orchestration | recursively **dispatch subagents** along the overview's drill-down sections to build/refresh the whole WHAT tree — **parallel fan-out** |
| **`arch-why-orchestrate`** | WHY orchestration | scan the tree for `⏳`, order them (foundations first), drive `arch-why-elicit` to **interview the human**, cross-check rationale consistency — **sequential, human-in-the-loop** |
| **`arch-spec-flow`** | SDD conductor | weave the two architecture touchpoints into a spec-driven flow: declare boundary-crossing intent in the spec up front, reconcile documents from landed code at branch wrap-up |

**Markers** used throughout the documents: `🧱` structural convention (hard) · `💡` recommended (has rationale) · `🚧` tentative · `⏳` pending (WHAT written, WHY/marker awaiting the human).

## Key ideas

### WHAT / WHY ownership split
- **WHAT** (structural facts) is written by `build`/`update` (analyze code, decompose components).
- **WHY + `🧱/💡/🚧` markers** are written and owned by the **human**, captured through `arch-why-elicit` by questioning and grounded bidirectionally against code to prevent fabrication.
- **`⏳`** is the state where WHAT is written but the marker/WHY are pending; in a boundary cell, `⏳` occupies the **same slot** as the marker.

### Why `build` and `update` are separate skills
They have **opposite disciplines**: `build` is a generative "fill the blanks" operation, while `update` is conservative "reconcile + minimal patch + preserve human WHY". Merging them puts two conflicting instruction sets in context at once and drifts at runtime; under an additive bias, a merged skill tends to degenerate into "regenerate" and overwrite the human-written WHY. Split, each skill has a single, non-conflicting goal — the most stable shape for an LLM.

### Documents trail code; review is the backstop
Documents record the architecture **as it is now** — no history, no process notes. A document is only ever written or updated **from landed code**, never ahead of it. Review uses a **strict completeness gate**: every document it consumes must be entirely `⏳`-free, or it halts — a forcing function against cut corners and stale documents. Review is **read-only** — it checks and reports, but changes nothing. Deciding *what* document change a finding warrants is `arch-doc-reconcile`'s job — also **read-only**, it produces a **suggestion report**; a human adopts the suggestions, and `arch-doc-update` / `arch-doc-build` carry them out. The whole set is **script-free and agent-enforced**; only the optional `enforced_by` hook is a per-item deterministic safeguard (attached only when a real static/dynamic check exists).

## Closed loop

```
Project bootstrap:  arch-doc-orchestrate  (fan out build/update → whole WHAT tree, all ⏳)
                          ▼
                    arch-why-orchestrate  (interview the human, drive arch-why-elicit → fill WHY, cross-doc consistency)
                          ▼
                    Document tree ⏳-free  (WHAT + WHY complete)

Per spec (SDD), via arch-spec-flow:
   spec finalization ─▶ declare any boundary-crossing intent IN THE SPEC (doc-path + INV-id)
   code lands        ─▶ arch-spec-review  (completeness gate → drift / consistency check against the declaration; read-only report)
   branch wrap-up    ─▶ arch-doc-reconcile suggests doc changes → human adopts → arch-doc-update / arch-doc-build write FROM LANDED CODE → arch-why-elicit fills new ⏳
                        (review + reconcile are read-only, run on the best model)
   merge             ─▶ only after documents are reconciled
```

## Repository layout

```
sdd-arch-skill/
├── skills/                       # the 9 skills (this is what gets installed)
│   ├── arch-docs-conventions/    # shared home: assets/template.md + references/intent-contract.md
│   ├── arch-doc-build/  ·  arch-doc-update/  ·  arch-why-elicit/  ·  arch-spec-review/  ·  arch-doc-reconcile/
│   ├── arch-doc-orchestrate/  ·  arch-why-orchestrate/  ·  arch-spec-flow/
├── .claude-plugin/   .codex-plugin/   .cursor-plugin/   .kimi-plugin/   # per-runtime manifests
├── package.json   LICENSE   README.md
```

(`_baseline/` holds private validation artifacts and is git-ignored — it is not part of the distribution.)

## License

MIT — see [LICENSE](LICENSE).
