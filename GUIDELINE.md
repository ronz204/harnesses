# Working with the Delta Harness

This is a guide for people, not agents — how to actually use this harness day to day. For the mechanics each skill follows internally, see the `SKILL.md` files under `.claude/skills/` (or `.agents/skills/`, `.cursor/skills/`, depending on your tool).

## What it is

The delta methodology keeps a project's documentation and specs grounded in the real codebase instead of drifting away from it. Instead of one big wiki that goes stale, you get:

- A **root orientation file** (`CLAUDE.md` / `AGENTS.md`) — short, just enough to get a session oriented and pointed at the right deeper file.
- **Docs** (`.claude/docs/*.md`) — durable, project-wide reference: architecture, module behavior, infrastructure. Read-once, cross-project knowledge.
- **Rules** (`.claude/rules/*.md`) — conventions that auto-load whenever a matching file is opened, so an agent doesn't have to be reminded every time.
- **Subagents** (`.claude/agents/*.md`) — bounded, repeatable tasks with restricted tool access and their own context window.
- **Deltas** (`services/<service>/deltas/*.md`) — the living contract for one slice of the product. This is the part that's specific to *your* project and where most of the day-to-day work happens.

Four skills operate this system: `surveyor` bootstraps it, `specifier` interviews to gather what a delta needs, `archivist` writes every artifact, `sentinel` checks whether a delta still matches reality. You never write these files by hand — you trigger a skill and it does the reading/writing for you.

## What a "delta" is

A **slice** is one cohesive capability — a feature, a flow, a bounded piece of the product (e.g. `checkout-flow`, not "the whole cart module" and not "the button that says Buy"). Each slice gets up to three flat files under `services/<service>/deltas/`:

| File | Layer | Contains |
|---|---|---|
| `<slice>.spec.md` | Logic | Intent, contract, invariants, deferred questions, acceptance criteria |
| `<slice>.design.md` (optional) | Presentation | Layout, states, interactions, what it consumes from `spec.md` — only for slices with a UI-facing surface |
| `<slice>.plan.md` (optional) | Roadmap | Step-by-step implementation plan to realize the spec/design |

**`spec.md` and `design.md` are living documents** — always describing what's true *now*, edited in place, never a history log. There's no changelog layer in this system on purpose: if something changed, the file is updated to reflect the new truth, not appended to. `plan.md` is the one exception — once every step lands, it's done its job and can be frozen or deleted.

A slice can have siblings in more than one service (e.g. a backend's `spec.md` for data rules and a frontend's own `spec.md` for the same slice name, scoped to client-side logic) — this is deliberate, not duplication, as long as each file states clearly what it owns.

## The four skills

| Skill | Job | Reads | Writes |
|---|---|---|---|
| `surveyor` | Bootstrap a project's knowledge base from scratch | The real repo (or interviews you, if greenfield) | `CLAUDE.md`/`AGENTS.md`, initial `docs/*` |
| `specifier` | Gather what a spec/design needs — intent, invariants, states, open questions | Existing spec/design, the real implementation, project docs/rules | Nothing — hands material to `archivist` |
| `archivist` | Write or update any artifact in the correct shape | The real source, existing docs/rules, `specifier`'s handoff | `CLAUDE.md`, docs, rules, agents, skills, `spec.md`/`design.md`/`plan.md` |
| `sentinel` | Check whether a delta's claims still hold against the real implementation | The delta's spec/design/plan, the real code | Nothing — reports, never edits |

`specifier` and `sentinel` never write anything themselves — they investigate and either interview you or verify, then hand off to `archivist` (or hand a correction back to `archivist`) for the actual write. This split keeps "did we ask the right question" separate from "does the file land in the right shape."

## Typical workflow

**1. Starting a brand-new project (or one with no knowledge base yet)**
Trigger `surveyor` — "bootstrap this repo", "monta la knowledge base". It scans the codebase (or interviews you if there's nothing to scan), asks a short batch of questions about vision/scope/infrastructure, and hands off to `archivist` to produce `CLAUDE.md` and whatever `docs/*` categories the project actually warrants. It also surfaces slice candidates — capabilities worth specifying later — without specifying any of them itself.

**2. Specifying a slice**
Trigger `specifier` — "vamos a especificar X", "hagamos el spec de este slice". It reads the real implementation, checks it against existing docs/rules, and asks you the things code alone can't answer: why the slice exists, what's explicitly out of scope, what must never break (invariants), what's intentionally undecided. If the slice has a UI, it asks the design-side questions too (states, interactions, what it consumes). It then hands the material to `archivist`, which writes `<slice>.spec.md` (and `<slice>.design.md`, if relevant) using the fixed templates.

**3. Implementing**
If the work is nontrivial, `archivist` can also draft `<slice>.plan.md` — an ordered, checkable roadmap tied to the spec/design's contract. You implement against that plan; nothing about this step is agent-magic, it's normal engineering work grounded in a contract that's now written down.

**4. Verifying**
After implementation (or any time you want a gut-check), trigger `sentinel` — "check this slice for drift", "¿sigue vigente este spec?". It reads the slice's spec/design/plan, extracts every falsifiable claim, and checks each one against the real code — cheaply, with targeted greps and delegated subagent reads rather than pulling everything into one context. It reports a table of Holds/Violated/Stale/Unverified and stops there; it never edits anything itself.

**5. Correcting drift**
If `sentinel` finds the *code* is wrong, that's a bug — fix the code. If it finds the *spec/design/plan* is wrong (the invariant no longer applies, a step's status is stale), that goes back through `archivist` to correct the file in place — never a silent pick-a-side.

**6. Everyday doc/rule updates**
Anything else — "documenta esta decisión", "turn this into a skill", "actualiza el CLAUDE.md" — triggers `archivist` directly. It always reads the real source first; it never writes from memory or from what was just said in chat.

## Which folder is "yours"

| You're working in... | Use |
|---|---|
| Claude Code | `.claude/` (canonical source) |
| Antigravity / Gemini CLI | `.agents/` |
| Cursor | `.cursor/` |

All three implement the same methodology; only file paths, extensions, and a few tool-specific mechanics (e.g. how a rule's frontmatter triggers) differ. `AGENTS.md` at the repo root is read natively by both Antigravity and Cursor as the shared entry point. `.claude/` is the one to edit first when the methodology itself changes — the other two get updated by hand to match.

## A few things worth knowing going in

- **Nothing gets written from memory.** Every skill re-reads the actual source before touching a file, even if you just described the change in chat.
- **No decision log, anywhere.** Specs, designs, docs, and rules describe current truth, not history. If you want "why did we change X," that lives in your commit history or PR descriptions, not in this system.
- **A rule fires on every future touch of a matching file.** That's why `archivist` always confirms a rule candidate with you before creating one — a false positive is more expensive than a missing doc.
- **Trivial edits skip the ceremony.** A typo fix or an already-settled one-line change doesn't need the full skill pipeline — the bar is whether the content is actually known and uncontested, not the size of the diff.
