# Delta Harness

Source repository for the delta methodology: a portable knowledge-base system built around four skills (`surveyor`, `specifier`, `archivist`, `sentinel`) that keep a project's specs, docs, and rules grounded in its real implementation instead of drifting from it. This repo holds the methodology's source; it is not itself a product with a `services/*/deltas/` of its own.

This file is the shared entry point for every tool that reads a root `AGENTS.md` natively (Antigravity/Gemini CLI, Cursor). The methodology's canonical source lives under `.claude/` (Claude Code); `.agents/` and `.cursor/` are mirrors adapted for Antigravity/Gemini CLI and for Cursor respectively — same methodology, same skill steps, ported terminology, file paths, and (where a tool's mechanism genuinely differs, e.g. Cursor's `.mdc` rule frontmatter) ported mechanics. Keep all trees in sync by hand: a change to a `.claude/skills/*` skill's actual behavior should be ported into its `.agents/skills/*` and `.cursor/skills/*` counterparts, not left to drift.

---

## Knowledge base layout

| Path | Holds |
|---|---|
| `AGENTS.md` | This file |
| `.agents/docs/`, `.cursor/docs/` | Self-contained reference files describing a downstream project's own system — empty here, since this repo has no product beyond the skills themselves |
| `.agents/rules/`, `.cursor/rules/` | Conventions loaded when a matching file is opened/edited — glob-scoped, per each tool's own rule-file mechanism |
| `.agents/agents/`, `.cursor/agents/` | Bounded, repeatable subagent tasks with their own tool access — empty here, same reason as the `docs/` rows |
| `.agents/skills/`, `.cursor/skills/` | The delta methodology itself, ported per tool: `surveyor` (bootstrap), `specifier` (interview), `archivist` (write), `sentinel` (verify), each with `SKILL.md` and (for `archivist`) its artifact templates under `references/` |
| `services/<service>/deltas/` | Where a service that adopts this harness keeps its per-slice `spec.md`/`design.md`/`plan.md` files (spec/design each carry an optional free-form Context section) — a layout convention this repo defines, not one it uses on itself |

## Repo layout

| Path | Purpose |
|---|---|
| `.claude/` | The methodology's canonical source, for Claude Code: skills, docs, rules, settings, MCP config |
| `.agents/` | The same methodology mirrored for Antigravity/Gemini CLI |
| `.cursor/` | The same methodology mirrored for Cursor |

## Setup & common commands

None. This repo has no build, install, or test step of its own — it's a source of agent-harness configuration (skills, templates, rules) meant to be copied into another project's `.claude/`, `.agents/`, and/or `.cursor/` directory, not run standalone.

## Permissions

Each tool's permission/safety configuration, if any is added, lives in that tool's own project-level config — check the current docs for the tool in question. Nothing has been ported here from `.claude/settings.json`, since permission policy is product-specific rather than part of the portable methodology.

## Conventions

- The mirrored trees are not required to be byte-identical — file paths, extensions, frontmatter fields, and a few tool-specific mentions (e.g. how a question gets asked, how a subagent gets dispatched) are adapted per tool — but the *methodology* (what each skill does, in what order, handing off to which other skill) must stay the same across all of them.
- `AGENTS.md` lives at the repo root; `.claude/CLAUDE.md` lives nested under `.claude/` — check the right one before assuming a path when orienting in Claude Code specifically.

---

## Non-goals

- This repo is the methodology's source, not a project that uses it — never create a `services/*/deltas/` here to document the skills themselves; they document themselves via their own `SKILL.md`.
- Not an application codebase — there is no product source to document under any tool's `docs/` directory.
