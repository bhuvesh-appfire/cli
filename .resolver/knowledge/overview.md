---
name: overview
description: Repo purpose and where to find the primary agent guidance for github/cli (gh)
type: knowledge
scope: global
updated: 2026-08-03 (IONE-959)
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - AGENTS.md
  - README.md
  - docs/project-layout.md
  - go.mod
---

`gh` is the GitHub CLI, module `github.com/cli/cli/v2`, Go 1.26. It is a single Go binary
(`cmd/gh/main.go`) exposing GitHub (GraphQL + REST) as a set of Cobra commands under `pkg/cmd/`.

The primary, already-thorough agent guidance lives in [`AGENTS.md`](../../AGENTS.md) — build/test/lint
commands, the Options+Factory command pattern, HTTP mocking, error types, feature detection, and code
style are all covered there. Read it first. [`docs/project-layout.md`](../../docs/project-layout.md)
explains the historical top-level packages (`api/`, `context/`, `git/`, `test/`, `utils/`) and how a
request flows from `main()` to a command's `RunE`.

This overlay only adds what those files don't: security-sensitive conventions enforced by custom CodeQL
queries (see [[security-conventions]]), the extension dispatch mechanism (see [[extensions]]), and the
overall component shape (see [[architecture]]).

## Gaps AGENTS.md doesn't cover

- **Terminal output sanitization** — `pkg/iostreams.Untrusted` / `IOStreams.ContentOut` exist specifically
  to stop server-controlled bytes from injecting ANSI escape sequences into the user's terminal. Not
  mentioned in AGENTS.md at all; enforced by `.github/codeql/queries/unsanitized-response-to-terminal.ql`.
  See [[security-conventions]].
- **REST path construction** — `internal/safeurl` builds percent-encoded REST paths so caller-supplied
  values can't smuggle extra path segments. Enforced by `.github/codeql/queries/SafeURLPathConstruction.ql`
  and `ImmutableSafeURLConstruction.ql`. See [[security-conventions]].
- **Extension dispatch** — `gh <extension-name>` is resolved and exec'd through
  `pkg/cmd/extension.Manager`, wired into the root command via `pkg/cmd/root/extension.go`. Not covered by
  AGENTS.md's command-structure section, which is about first-party `pkg/cmd/<command>` packages only. See
  [[extensions]].
- **CI test invocation differs slightly from AGENTS.md** — `.github/workflows/go.yml` runs
  `go test -race -tags=integration ./...`, not the plain `go test ./...` shown in AGENTS.md. There's also a
  separate `integration-tests` job that shells out to `test/integration/attestation-cmd/run-all-tests.sh`
  against a real `GH_TOKEN`. Acceptance tests (`go:build acceptance`) are documented in depth in
  [`acceptance/README.md`](../../acceptance/README.md) and require `GH_ACCEPTANCE_HOST`/`_ORG`/`_TOKEN`
  against a real GitHub instance — don't invent a way to run them without those.
