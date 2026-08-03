---
name: overview
description: What this repo is, where the real docs live, and which top-level packages are dead weight
type: knowledge
scope: global
updated: 2026-08-03 (IONE-959)
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - AGENTS.md
  - docs/project-layout.md
  - go.mod
---

This is `gh`, the GitHub CLI (module `github.com/cli/cli/v2`, Go 1.26).

The primary agent-facing reference is [`AGENTS.md`](../../AGENTS.md) — it already covers build/test/lint commands, the entry-point chain, the Options+Factory command pattern, JSON output flags, HTTP mocking, error types, and feature-detection conventions in detail. Read it first; this overlay only adds what it doesn't cover.

For directory-level orientation, [`docs/project-layout.md`](../../docs/project-layout.md) is accurate and worth citing directly rather than restating.

## Dead / deprecated top-level packages

`docs/project-layout.md` flags these, and the code backs it up — don't add new code to them:
- `context/` — deprecated, git-remote lookups only
- `test/` — deprecated, do not use
- `utils/` — deprecated, table-output printing only

New shared logic belongs in `internal/` (if `gh`-specific) or a scoped `pkg/cmd/<command>/` package, per `docs/project-layout.md`'s "how to add a new command" guidance.

## Agent skills surface

`skills/gh/SKILL.md` and `skills/gh-skill/SKILL.md` ship an installable [agent skill](https://agentskills.io) via `gh skill install`/`gh skill update` (see `README.md`). This is a distinct concern from `internal/agents/detect.go`, which detects *which* coding agent is driving `gh` itself (for telemetry dimensions) — see [[gotchas]] for how that detection is ordered.
