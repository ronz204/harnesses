# Delta Harness

Source repository for the delta methodology: a portable knowledge-base system built around four skills (`surveyor`, `specifier`, `archivist`, `sentinel`) that keep a project's specs, docs, and rules grounded in its real implementation instead of drifting from it. This repo holds the methodology's source; it is not itself a product with a `services/*/deltas/` of its own.

This file is the Antigravity-side entry point. The methodology's canonical source lives under `.claude/` (Claude Code); everything under `.agents/` is a mirror adapted for Antigravity/Gemini CLI — same methodology, same skill steps, ported terminology and file paths. Keep the two trees in sync by hand: a change to a `.claude/skills/*` skill's actual behavior should be ported into its `.agents/skills/*` counterpart, not left to drift.

---

## Knowledge base layout

| Path | Holds |
|---|---|
| `AGENTS.md` | This file |
| `.agents/docs/` | Self-contained reference files describing a downstream project's own system — empty here, since this repo has no product beyond the skills themselves |
| `.agents/rules/` | Conventions auto-loaded when a matching file is opened/edited, scoped via `paths:` frontmatter |
| `.agents/agents/` | Bounded, repeatable subagent tasks with their own tool access — empty here, same reason as `.agents/docs/` |
| `.agents/skills/` | The delta methodology itself, ported for Antigravity: `surveyor` (bootstrap), `specifier` (interview), `archivist` (write), `sentinel` (verify), each with `SKILL.md` and (for `archivist`) its artifact templates under `references/` |
| `services/<service>/deltas/` | Where a service that adopts this harness keeps its per-slice `spec.md`/`design.md`/`plan.md` files (spec/design each carry an optional free-form Context section) — a layout convention this repo defines, not one it uses on itself |

## Repo layout

| Path | Purpose |
|---|---|
| `.claude/` | The methodology's canonical source, for Claude Code: skills, docs, rules, settings, MCP config |
| `.agents/` | The same methodology mirrored for Antigravity/Gemini CLI |

## Setup & common commands

None. This repo has no build, install, or test step of its own — it's a source of agent-harness configuration (skills, templates, rules) meant to be copied into another project's `.claude/` and/or `.agents/` directory, not run standalone.

## Permissions

Antigravity's permission/safety configuration, if any is added, would live in its own project-level config file — check current Antigravity docs for where that is; nothing has been ported here from `.claude/settings.json`, since permission policy is product-specific rather than part of the portable methodology.

## Conventions

- The two trees are not required to be byte-identical — file paths and a few tool-specific mentions (e.g. how a question gets asked, how a subagent gets dispatched) are adapted per tool — but the *methodology* (what each skill does, in what order, handing off to which other skill) must stay the same in both.
- `AGENTS.md` lives at the repo root; `.claude/CLAUDE.md` lives nested under `.claude/` — check the right one before assuming a path when orienting in either tool.

---

## Non-goals

- This repo is the methodology's source, not a project that uses it — never create a `services/*/deltas/` here to document the skills themselves; they document themselves via their own `SKILL.md`.
- Not an application codebase — there is no product source to document under `.agents/docs/` or `.claude/docs/`.
