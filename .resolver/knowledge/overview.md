---
name: overview
description: What this repo is and where the authoritative agent guidance already lives
type: knowledge
scope: global
updated: '2026-08-05'
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - AGENTS.md
  - docs/project-layout.md
  - go.mod
---

This is `github.com/cli/cli/v2`, the GitHub CLI (`gh`) - a single Go module producing the `gh` binary.

Start with [`AGENTS.md`](../../AGENTS.md) for build/test/lint commands, the Options+Factory command pattern, JSON output flags, HTTP mocking, error types, and feature-detection conventions - it is thorough and current, so this overlay does not restate it. [`docs/project-layout.md`](../../docs/project-layout.md) covers directory layout and the "how a command executes" walkthrough.

This overlay only adds what those files don't: command-resolution precedence and BaseRepo behavior ([[architecture]]), and non-obvious runtime behaviors ([[gotchas]]).

## Notable subsystems not called out in AGENTS.md

- `pkg/cmd/copilot/` - `gh copilot` downloads and execs the separate GitHub Copilot CLI binary (archive extraction, checksum verification), it does not implement Copilot itself.
- `pkg/cmd/agent-task/` - preview command for GitHub's async coding agent; subcommands require a device-flow OAuth token (see [[gotchas]]).
- `pkg/cmd/attestation/` - sigstore-based artifact attestation verification (`internal`/vendor deps: `sigstore-go`, `go-tuf`, `in-toto`).
- `internal/codespaces/rpc/` - generated gRPC/protobuf clients for the Codespaces connection server; regenerate via `internal/codespaces/rpc/generate.sh`, do not hand-edit `*.pb.go`/`*_grpc.pb.go`.
- `.github/skills/` and top-level `skills/` - Claude/agent skill definitions (`SKILL.md`) for code review, dependabot triage, and agentic-workflow routing; read the relevant `SKILL.md` before doing that kind of task.
- `context/`, `test/`, `utils/` at the repo root are marked DEPRECATED in `docs/project-layout.md` - don't add new code there.
