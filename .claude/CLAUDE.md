# Delta Harness

Source repository for the delta methodology: a portable Claude Code knowledge-base system built around four skills (`surveyor`, `specifier`, `archivist`, `sentinel`) that keep a project's specs, docs, and rules grounded in its real implementation instead of drifting from it. This repo holds the methodology's source; it is not itself a product with a `services/*/deltas/` of its own.

---

## Knowledge base layout

| Path | Holds |
|---|---|
| `.claude/CLAUDE.md` | This file |
| `.claude/docs/` | Self-contained reference files describing a downstream project's own system — empty here, since this repo has no product beyond the skills themselves |
| `.claude/rules/` | Conventions auto-loaded when a matching file is opened/edited, scoped via `paths:` frontmatter |
| `.claude/skills/` | The delta methodology itself: `surveyor` (bootstrap), `specifier` (interview), `archivist` (write), `sentinel` (verify), each with `SKILL.md` and (for `archivist`) its artifact templates under `references/` |
| `.claude/settings.json` | Permission policy — see Permissions below |
| `.claude/.mcp.json` | Project-scoped MCP server config |
| `services/<service>/deltas/` | Where a service that adopts this harness keeps its per-slice `spec.md`/`design.md`/`plan.md` files (spec/design each carry an optional free-form Context section) — a layout convention this repo defines, not one it uses on itself |

## Repo layout

| Path | Purpose |
|---|---|
| `.claude/` | Everything above: the methodology's own skills, docs, rules, settings, MCP config |

## Setup & common commands

None. This repo has no build, install, or test step of its own — it's a source of Claude Code configuration (skills, templates, rules) meant to be copied into another project's `.claude/` directory, not run standalone.

## Permissions

The full policy lives in `.claude/settings.json`. Destructive git operations (`push`, `pull`) and reads of `.env`/secrets files are denied by default; read-only inspection commands (`git status`/`log`/`diff`/`show`, `curl`, `docker exec`/`compose`) and `WebSearch` are pre-approved.

## Conventions

`CLAUDE.md` lives under `.claude/` in this repo, not at the root — check here before assuming a root-level file when orienting. Any edit to this file, `.claude/rules/`, `.claude/agents/`, or `.claude/skills/` routes through `archivist` rather than being hand-edited — see `.claude/rules/delta-artifacts.md`.

---

## Non-goals

- This repo is the methodology's source, not a project that uses it — never create a `services/*/deltas/` here to document the skills themselves; they document themselves via their own `SKILL.md`.
- Not an application codebase — there is no product source to document under `.claude/docs/`.
