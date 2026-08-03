# .resolver index

## knowledge

- `knowledge/architecture.md` · global · How a gh invocation flows through main -> root -> command -> api/git/extension, as a diagram
- `knowledge/extensions.md` · pkg/cmd/extension/**, pkg/cmd/root/extension.go, pkg/cmd/root/root.go · How gh extensions are discovered, installed, and dispatched - read before touching extension code
- `knowledge/overview.md` · global · Repo purpose and where to find the primary agent guidance for github/cli (gh)

## conventions

- `conventions/ingested-agents.md` · global · Team agents-md rules from AGENTS.md
- `conventions/security-conventions.md` · global · Mandatory patterns for REST path construction and printing external content - required in new command code
- `conventions/testing-and-ci.md` · global · How CI actually invokes tests, beyond the `go test ./...` shown in AGENTS.md
