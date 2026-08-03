2026-08-03 · first-run · created .resolver

- Indexed AGENTS.md, docs/project-layout.md, .github/CONTRIBUTING.md, acceptance/README.md, .golangci.yml, go.mod, and core entrypoint/factory/api code to verify claims.
- AGENTS.md is already comprehensive (build/test/lint, entrypoint chain, Options+Factory pattern, HTTP mocking, error types, feature-detection convention) — no divergences found; this overlay only adds gaps: golangci-lint's actual enabled-linter allowlist, the acceptance/testscript suite's relationship to unit tests, and runtime gotchas (Factory's three HTTP clients, config eager-load, telemetry GHES auto-disable, agent-detection ordering) not written down anywhere in the customer repo.
- No database/schema found in the repo (it's a CLI tool), so knowledge/data-model.md was skipped per instructions.
