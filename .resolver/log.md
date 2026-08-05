2026-08-05 · first-run · created .resolver

- Read AGENTS.md, docs/project-layout.md, .github/CONTRIBUTING.md, acceptance/README.md, docs/multiple-accounts.md, docs/macos-keyring.md, docs/releasing.md - all already thorough and current, so cited rather than duplicated.
- Captured gaps not covered by those docs: command-resolution precedence (core > extensions > aliases > official stubs), the two BaseRepo resolution strategies (plain vs Smart), telemetry/auth-check annotation wiring, ghinstance host special-casing (garage.github.com, github.localhost), agent-task's device-flow-OAuth-only auth gate, and the fact that errcheck/staticcheck/gosec are intentionally disabled in .golangci.yml.
- No database/schema found in the repo, so skipped knowledge/data-model.md.
