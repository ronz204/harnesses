### Harnesses for LLMs

Source repository for the **delta methodology**: a portable knowledge-base system built around four skills — `surveyor` (bootstrap), `specifier` (interview), `archivist` (write), `sentinel` (verify) — that keep a project's specs, docs, and rules grounded in its real implementation instead of drifting from it.

This repo *is* the methodology's source. It has no application code and no build/test step of its own — it's meant to be copied into another project's agent-config directory.

## Layout

| Path | What it is |
|---|---|
| `.claude/` | Canonical source, for Claude Code |
| `.agents/` | Mirror adapted for Antigravity / Gemini CLI |
| `.cursor/` | Mirror adapted for Cursor |
| `AGENTS.md` | Shared root entry point, read natively by Antigravity and Cursor |

All three trees implement the same methodology — same skills, same steps, same handoffs — just adapted to each tool's own file paths and rule mechanics. `.claude/` is the one to edit first; changes get ported into the other two by hand.

See `AGENTS.md` (or `.claude/CLAUDE.md`, for Claude Code specifics) for the full knowledge-base layout and conventions.
