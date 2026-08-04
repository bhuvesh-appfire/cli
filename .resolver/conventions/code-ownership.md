---
name: code-ownership
description: Which subtrees require a non-default reviewer team - check before proposing changes there
type: convention
scope: global
updated: '2026-08-04'
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - .github/CODEOWNERS
---

Default reviewer team for the whole repo is `@cli/code-reviewers`. The following subtrees have an *additional* required team on top of that default (from `.github/CODEOWNERS`):

| Path | Additional team |
|---|---|
| `pkg/cmd/codespace/`, `internal/codespaces/` | `@cli/codespaces` |
| `pkg/cmd/attestation/`, `pkg/cmd/release/verify/`, `pkg/cmd/release/verify-asset/`, `pkg/cmd/release/shared/`, `test/integration/attestation-cmd` | `@cli/package-security` |
| `pkg/cmd/attestation/verification/embed/tuf-repo.github.com/` | `@cli/tuf-root-reviewers` |
| `pkg/cmd/skills/`, `internal/skills/` | `@cli/skills` |

When a change touches one of these paths, flag it explicitly - it signals the change is in a domain with extra review requirements (security-sensitive attestation/TUF verification, or codespaces-specific integration surface), not just an ordinary command package. This complements [.github/CONTRIBUTING.md](../../.github/CONTRIBUTING.md)'s rule against opening PRs for issues labelled `core`.
