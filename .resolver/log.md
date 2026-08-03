2026-08-03 · first-run · created .resolver

- Indexed AGENTS.md (already comprehensive: build/test/lint, Options+Factory pattern, HTTP mocking,
  error types, feature detection, code style) and docs/project-layout.md; overlay focuses on gaps only.
- Added knowledge/overview.md, knowledge/architecture.md (mermaid), knowledge/extensions.md.
- Added conventions/security-conventions.md (safeurl REST path construction, IOStreams.ContentOut/
  Untrusted terminal-sanitization, gosec exclusions) and conventions/testing-and-ci.md (CI test surfaces
  beyond AGENTS.md's `go test ./...` example).
- No divergences found that contradict AGENTS.md's claims outright; all overlay content covers gaps.
