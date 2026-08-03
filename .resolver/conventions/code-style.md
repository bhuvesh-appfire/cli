---
name: code-style
description: Lint enforcement specifics not covered by AGENTS.md's Code Style section
type: convention
scope: global
updated: 2026-08-03 (IONE-959)
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - .golangci.yml
  - AGENTS.md
---

`AGENTS.md`'s "Code Style" section covers commenting rules (godoc on exports, comment only for non-obvious *why*, no em dashes). This page adds what `make lint` actually enforces, since the linter config is stricter/narrower than "standard golangci-lint defaults" would suggest.

## Enabled linters (`.golangci.yml`)

Only an explicit allowlist runs (`linters.default: none`): `asasalint`, `asciicheck`, `bidichk`, `bodyclose`, `copyloopvar`, `durationcheck`, `exptostd`, `fatcontext`, `gocheckcompilerdirectives`, `gochecksumtype`, `gocritic` (style tag disabled, `appendAssign` disabled), `gomoddirectives`, `goprintffuncname`, `govet` (+`httpresponse`), `ineffassign`, `nilerr`, `nolintlint`, `nosprintfhostport`, `reassign`, `unused`.

**`gosec`, `staticcheck`, and `errcheck` are explicitly NOT enabled yet** (commented "to enable later due to too many issues"). Don't assume unchecked errors or common security footguns will be caught by CI — review those manually, and don't rely on `//nolint:errcheck` etc. being meaningful since the linter isn't active.

`bodyclose` and `gosec` are excluded for `_test.go` files.

## ASCII / unicode

`asciicheck` and `bidichk` are both enabled — don't introduce non-ASCII identifiers or bidirectional-override unicode sequences anywhere in source, not just user-facing strings.
