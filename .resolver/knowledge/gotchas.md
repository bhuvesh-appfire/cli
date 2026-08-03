---
name: gotchas
description: Non-obvious runtime behaviors around Factory HTTP clients, config error handling, telemetry, and agent detection
type: knowledge
scope: global
updated: 2026-08-03 (IONE-959)
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - pkg/cmdutil/factory.go
  - internal/ghcmd/cmd.go
  - internal/agents/detect.go
  - internal/featuredetection/feature_detection.go
---

## Three different HTTP clients on Factory, not one

`pkg/cmdutil/factory.go` exposes `HttpClient`, `PlainHttpClient`, and `ExternalHttpClient` as distinct fields, each with a documented, non-interchangeable purpose:
- `HttpClient` — attaches auth tokens and GitHub-specific headers automatically. Use this for talking to the GitHub API in the common case.
- `PlainHttpClient` — no automatic headers; used only where the caller needs to set headers itself (e.g. during login flows).
- `ExternalHttpClient` — for non-GitHub hosts; adds debug logging and a User-Agent but no GitHub auth.

Picking the wrong one either leaks no auth where it's needed, or attaches GitHub auth headers to a third-party request.

## Config is eagerly loaded and never fails startup

`internal/ghcmd.Main()` calls `config.NewConfig()` once and captures both the config and its error in a closure (`cfgFunc`), rather than propagating the error immediately. This is deliberate (see the comment on `Factory.Config` in `pkg/cmdutil/factory.go`) so that commands like `gh version` still run even with a broken config. Any command that needs config must check the error itself when calling `opts.Config()` — a nil/broken config is a real code path, not a startup invariant.

## Telemetry silently disables itself for likely-GHES users

`mightBeGHESUser()` in `internal/ghcmd/cmd.go` force-disables telemetry (regardless of the user's telemetry setting) if `GH_ENTERPRISE_TOKEN`/`GITHUB_ENTERPRISE_TOKEN` is set, `GH_HOST` points at an enterprise host, or any configured auth host is an enterprise host. If you're debugging "why isn't telemetry firing," check these before assuming the telemetry service itself is broken.

## Agent-detection env-var checks are order-dependent

`internal/agents.Detect()` (`internal/agents/detect.go`) checks environment variables in a specific, commented order because several coding agents set overlapping env vars (e.g. Amp and Cowork both set variables that a more generic check would also match). If you add detection for a new agent, you must insert the check at the correct specificity rank, not append it at the end — read the existing comments explaining why each check is ordered where it is.

## Feature-detection TODO comments are load-bearing, not decorative

`AGENTS.md` already documents the `// TODO <cleanupIdentifier>` convention above feature-detection if-statements. In this codebase those identifiers are also used to track GHES-support-window cleanup (e.g. `// TODO IssueRelationshipsCleanup - remove when GHES 3.18 support ends (~October 2026)` in `internal/featuredetection/feature_detection.go`). When adding a new feature flag, include the removal condition/date in the comment, not just the identifier — it's how the team finds dead feature-detection code later.
