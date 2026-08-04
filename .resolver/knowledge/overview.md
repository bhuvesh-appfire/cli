---
name: overview
description: What this repo is and where the primary agent guidance already lives - read this first
type: knowledge
scope: global
updated: '2026-08-04'
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - AGENTS.md
  - README.md
  - docs/project-layout.md
  - go.mod
---

This is `gh`, the official GitHub CLI, module `github.com/cli/cli/v2` (`go.mod:1,3`). It targets Go 1.26 (toolchain 1.26.5).

Primary guidance already lives in the repo and should be treated as authoritative first:
- [`AGENTS.md`](../../AGENTS.md) - build/test/lint commands, architecture entry point, the Options+Factory command pattern, JSON output flags, HTTP mocking, error types, feature-detection convention, code style. This overlay does not restate any of it.
- [`docs/project-layout.md`](../../docs/project-layout.md) - top-level directory map (`cmd/`, `pkg/`, `internal/`, `api/`, `git/`, `context/`).
- [`.github/CONTRIBUTING.md`](../../.github/CONTRIBUTING.md) - contribution policy: only work issues labelled `help wanted`, do not expand PR scope beyond the issue's Acceptance Criteria, never open PRs against issues labelled `core`.

## Gaps this overlay fills

- [[architecture]] - a verified call-graph diagram from `cmd/gh/main.go` down through the factory and command tree (not diagrammed anywhere in the repo's own docs).
- [[code-ownership]] - which subtrees have a narrower reviewer set than the default `@cli/code-reviewers`, derived from `.github/CODEOWNERS`.
- [[lint-gotchas]] - `.golangci.yml` specifics AGENTS.md doesn't spell out: which common linters are deliberately *not* enabled yet, and the test-file exclusions.
- [[acceptance-testing]] - conventions for the `testscript`-based acceptance suite in `acceptance/`, which is a separate concern from the unit-test patterns AGENTS.md documents.

## Non-obvious fact: "deprecated" top-level packages are still load-bearing

`docs/project-layout.md` labels `test/` and `utils/` as deprecated ("use only for ...", "do not use" for `test/`). In practice `test/helpers.go` (package `test`) is still imported by 23+ existing `_test.go` files across `pkg/cmd/**` for `test.CmdOut`/`test.ExpectLines`, and `utils/` is still imported by 7 non-test files including `internal/ghcmd/cmd.go` and `api/http_client.go`. Deprecated here means "don't add new usages," not "dead code" - don't delete or "clean up" these packages as unused.
