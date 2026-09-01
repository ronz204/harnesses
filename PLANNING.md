# `delta init` — a Go CLI to bootstrap this harness into a new project

Status: planned, not started.

## Context

The harness currently gets adopted by hand: clone/copy `.claude/`, `.agents/`, and/or `.cursor/` from this repo into a target project. Now that `harnesses` is public on GitHub (`git@gh-personal:ronz204/harnesses.git` → `github.com/ronz204/harnesses`, **assumption to confirm** — the SSH alias may not map 1:1 to the GitHub username), the goal is a small Go CLI that does this automatically instead. Scope for v1: `init` only (no `update`/`status` yet), distributed via `go install`, supporting all three tool trees (`claude`, `agents`, `cursor`).

## What gets copied, and what doesn't

Portable methodology (safe to copy into any target project) vs. this-repo-specific content (must never land in a target project):

| Copy | Skip |
|---|---|
| `.claude/skills/**`, `.agents/skills/**`, `.cursor/skills/**` | `.claude/CLAUDE.md`, `AGENTS.md` (root) — self-description of *this* repo; target gets its own via `surveyor` |
| `.claude/rules/**`, `.agents/rules/**`, `.cursor/rules/**` (currently `delta-artifacts.md` on the Claude side only — the other two trees only have `.gitkeep`; that mirror gap is pre-existing and out of scope here) | `.claude/settings.json`, `.mcp.json` — this repo's own dev permissions/MCP config, not part of the methodology (confirmed by `AGENTS.md`: "Nothing has been ported here from `.claude/settings.json`, since permission policy is product-specific") |
| `.claude/agents/**`, `.agents/agents/**`, `.cursor/agents/**` (currently just `.gitkeep`) | `README.md`, `GUIDELINE.md` — human docs about the source repo, not harness artifacts |
| `.claude/docs/**`, `.agents/docs/**`, `.cursor/docs/**` (currently just `.gitkeep`) | `services/**/deltas/**` — doesn't exist in this repo anyway |

This lines up with `.claude/rules/delta-artifacts.md`'s own path list (`CLAUDE.md`, `docs/`, `rules/`, `agents/`, `skills/`) minus `CLAUDE.md` itself, which is uniquely self-referential.

## Design: embed at build time, no runtime fetch

Rejected a runtime fetch (tarball-over-HTTP or `git clone`) in favor of `go:embed`: since this Go tool *lives in the same repo* it's distributing, `go install github.com/ronz204/harnesses/cmd/delta@<ref>` already gives every user the payload frozen at that exact ref, with zero network/parsing code. Simpler, and the module version Go itself resolved becomes the version record for free via `runtime/debug.ReadBuildInfo()` — no need to hand-maintain a version string.

`go:embed` cannot reach outside the directory of the file containing the directive, so the embed lives in a small package at the repo root (sibling to `cmd/`), and must use the `all:` prefix per tree/subfolder, since embedded dot-files (`.gitkeep`) are excluded by default:

```go
// payload.go (package harnesses, repo root)
package harnesses

import "embed"

//go:embed all:.claude/skills all:.claude/rules all:.claude/agents all:.claude/docs
//go:embed all:.agents/skills all:.agents/rules all:.agents/agents all:.agents/docs
//go:embed all:.cursor/skills all:.cursor/rules all:.cursor/agents all:.cursor/docs
var Payload embed.FS
```

Embedding only the `skills/rules/agents/docs` subdirectories (never the tree root) is what naturally excludes `CLAUDE.md`/`settings.json`/`.mcp.json` — no manual exclude-list to keep in sync as the repo evolves.

## New files

- **`go.mod`** (repo root) — `module github.com/ronz204/harnesses`, `go 1.26`. First build artifact this repo has ever had; `README.md`/`.claude/CLAUDE.md`'s "Setup & common commands: None" line will need a follow-up update once this lands — the `CLAUDE.md` edit specifically routes through `archivist` per this repo's own rule, so that's a separate, later step, not part of this plan's execution.
- **`payload.go`** (repo root) — the embed directive above.
- **`cmd/delta/main.go`** — the CLI. Stdlib only (`flag`, `embed`/`fs`, `runtime/debug`, `crypto/sha256`, `encoding/json`), no third-party dependency, to keep `go install` fast and the tool genuinely "mini":
  - Flags: `--dir` (default `.`), `--tools` (comma list, default `claude,agents,cursor`), `--force` (overwrite files that differ locally).
  - For each selected tool, `fs.WalkDir` over `Payload` rooted at `.<tool>`, and for each file: target path is `<dir>/<relpath>` (paths line up 1:1, no rewriting). If the target exists and its content differs from the payload and `--force` isn't set, skip and report it as kept; otherwise write it (creating parent dirs as needed).
  - Resolve the installed ref via `debug.ReadBuildInfo().Main.Version` (what `go install pkg@ref` resolved to — e.g. `v0.1.0` or a pseudo-version).
  - Write `.delta-manifest.json` at `<dir>` root: `{"repo": "github.com/ronz204/harnesses", "ref": "<resolved>", "tools": [...], "installedAt": "<RFC3339>", "files": {"<relpath>": "<sha256 of written content>"}}`. Not consumed by anything yet — this is the seed a future `update` command will diff against, so it belongs in v1 even though `update` doesn't exist yet.
  - Print a short summary: files written vs. files kept (already existed, differing content), per tool.

## Verification

1. `go build ./...` at repo root — confirms `go.mod`/embed paths/package layout are all correct.
2. `go run ./cmd/delta init --dir <scratch dir>` — confirm `.claude/`, `.agents/`, `.cursor/` appear with `skills/` (with `SKILL.md` + `archivist/references/*`), `rules/`, `agents/`, `docs/`, and confirm `CLAUDE.md`, `AGENTS.md`, `settings.json`, `.mcp.json` are **absent**. Confirm `.delta-manifest.json` is written and its `ref`/`files` look right.
3. Re-run the same command against the same scratch dir — confirm it reports everything as already-present/kept, not duplicated or errored (idempotency).
4. Hand-edit one copied file (e.g. tweak `surveyor/SKILL.md` in the scratch dir) and re-run without `--force` — confirm that file is reported as kept/skipped rather than silently overwritten.
5. `go run ./cmd/delta init --dir <fresh scratch dir> --tools claude` — confirm only `.claude/` is written, no `.agents/`/`.cursor/`.

## Explicitly out of scope (follow-ups, not part of this plan)

- `delta update` / `delta status` — reading `.delta-manifest.json` back, fetching the latest ref, diffing local-modified vs. safe-to-overwrite files.
- Prebuilt binaries / GoReleaser / `curl | sh` installer — `go install` is the only distribution path for v1.
- Filling the `.agents/rules` and `.cursor/rules` mirror gap (no `delta-artifacts.md` equivalent there yet) — a separate harness-content fix, not a tool-code change, and would route through `archivist` per this repo's own convention.
- Updating `README.md`/`.claude/CLAUDE.md` to document the new install command.
