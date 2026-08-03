---
name: security-conventions
description: Mandatory patterns for REST path construction and printing external content - required in new command code
type: convention
scope: global
updated: 2026-08-03 (IONE-959)
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - internal/safeurl/safeurl.go
  - pkg/iostreams/untrusted.go
  - pkg/iostreams/iostreams.go
  - .github/codeql/queries/unsanitized-response-to-terminal.ql
  - .github/codeql/queries/SafeURLPathConstruction.ql
  - .github/codeql/queries/ImmutableSafeURLConstruction.ql
  - .golangci.yml
---

These two patterns are not mentioned in `AGENTS.md` but are enforced by custom CodeQL queries in
`.github/codeql/queries/` and checked in CI (`.github/codeql/codeql-config.yml`). Both apply to any new
command that hits a REST endpoint or prints response bytes.

## Building REST paths: use `internal/safeurl`, never `fmt.Sprintf`

Any REST path built from a variable (owner, repo, id, branch name, etc.) must go through
`safeurl.JoinPath(...)` or `safeurl.JoinPathWithHostPrefix(prefix, ...)`, not raw string
concatenation/`fmt.Sprintf`. This is the pattern used across ~90 files under `pkg/cmd/` and `api/`:

```go
path, err := safeurl.JoinPath("repos", owner, repo, "issues", strconv.Itoa(number))
if err != nil {
    return err
}
return client.REST(host, "GET", path.String(), nil, &data)
```

- `JoinPath`/`JoinPathWithHostPrefix` percent-encode every component and reject a literal `".."` component,
  so a caller-supplied value can't traverse the path or smuggle extra segments.
- `NewImmutableSafeURL(url)` is the escape hatch for URLs you did *not* build from components — a
  pagination `next` link, an asset URL from a server response. Only wrap URLs you already trust; it
  renders verbatim with no encoding. Don't use it to sidestep `JoinPath` for a URL you are constructing
  yourself from variables.

## Printing external content: `IOStreams.Out` vs `IOStreams.ContentOut`

`IOStreams.Out` never sanitizes — it's for output the developer authored (tables, prompts, formatted
messages) and must stay that way. `IOStreams.ContentOut` (`pkg/iostreams/iostreams.go:58-63`) is the writer
for content a third party chose the bytes for: HTTP response bodies, remote file contents. It sanitizes
ANSI escape sequences by default so a malicious server response can't repaint the user's terminal or hijack
the clipboard via OSC sequences.

Concretely:
- Label response bytes with `iostreams.NewUntrusted(s)` / `NewUntrustedBytes(b)` and print with `%s`/`%v`/
  `String()` — `Untrusted.String()` sanitizes automatically. Its `Raw()` method is an explicit, CodeQL-flagged
  opt-out.
- Or write the bytes directly to `opts.IO.ContentOut` instead of `opts.IO.Out`.
- Decoding bytes with `encoding/json` first (then printing the typed fields) also satisfies this — the
  values are no longer raw external content.
- If a command legitimately needs raw passthrough (e.g. dumping raw file content), add a per-command
  `--allow-escape-sequences` flag and call `opts.IO.SetContentSanitization(false)` before writing, per the
  pattern in `pkg/cmd/repo/read-file/read_file.go` and `pkg/cmd/gist/view/view.go`.

## gosec is intentionally narrowed

`.golangci.yml` disables `gosec` checks G110, G204, G301, G302, G304, G307 because `gh` legitimately shells
out with variable args (extensions, git) and reads/writes variable file paths (gist, release download,
repo read-file) as its normal job. Don't treat "gosec would flag this" as a blocker for those patterns —
but don't use the exclusion as license to skip the `safeurl`/`ContentOut` conventions above, which cover
the same risk classes more precisely.
