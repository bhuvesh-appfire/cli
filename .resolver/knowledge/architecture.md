---
name: architecture
description: How a gh invocation flows through main -> root -> command -> api/git/extension, as a diagram
type: knowledge
scope: global
updated: 2026-08-03 (IONE-959)
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - cmd/gh/main.go
  - internal/ghcmd/cmd.go
  - pkg/cmd/root/root.go
  - pkg/cmd/root/extension.go
  - pkg/cmd/factory/default.go
  - pkg/cmdutil/factory.go
  - pkg/cmd/extension/manager.go
  - api/client.go
  - git/client.go
  - context/context.go
---

```mermaid
flowchart TD
    Main["cmd/gh/main.go"] --> GhCmd["internal/ghcmd.Main()"]
    GhCmd --> Root["pkg/cmd/root.NewCmdRoot()"]
    Root --> Cmds["pkg/cmd/&lt;cmd&gt;/&lt;subcmd&gt; (cobra commands)"]
    Root --> ExtCmd["pkg/cmd/root/extension.go: NewCmdExtension"]
    ExtCmd --> ExtMgr["pkg/cmd/extension.Manager.Dispatch()"]
    ExtMgr --> ExtBin["external extension binary (os/exec)"]

    Factory["pkg/cmdutil.Factory\n(built by pkg/cmd/factory.New)"] --> Cmds
    Factory --> HTTPClient["api.Client\n(GraphQL + REST via go-gh)"]
    Factory --> GitClient["git.Client\n(shells out to local git)"]
    Factory --> Remotes["context.Remotes\n(resolves BaseRepo from git remotes)"]
    Factory --> Config["internal/config (gh.Config)\n~/.config/gh/{config,hosts}.yml"]
    Factory --> ExtMgr

    HTTPClient --> GitHubAPI[("GitHub.com / GHES REST + GraphQL")]
    GitClient --> LocalGit[("local .git repository")]
    Remotes --> GitClient
```

- `internal/ghcmd.Main()` is the thin, real entrypoint; `func main()` in `cmd/gh/main.go` just calls it and
  sets the process exit code (also exercised directly as the `gh` binary inside acceptance tests via
  `testscript.RunMain`, see `acceptance/acceptance_test.go`).
- `pkg/cmd/factory.New()` builds every dependency a command needs (`api`, `git`, config, prompter, browser,
  extension manager) as lazily-invoked closures on `cmdutil.Factory` — commands call e.g. `f.HttpClient()`
  inside `RunE`, never at construction time (see `pkg/cmd/factory/default.go:26-47`).
- Extensions are not `pkg/cmd/` packages. `pkg/cmd/root/extension.go` registers one cobra command per
  discovered extension and delegates actual execution to `extension.Manager.Dispatch`, which execs the
  extension's binary/git-checkout with the remaining args. See [[extensions]].
- `context.Remotes` / `BaseRepoFunc` resolve which repo a command targets from local git remotes filtered
  to authenticated hosts (`pkg/cmd/factory/default.go:49-82`); `docs/project-layout.md` calls `context/`
  itself deprecated but this resolution flow is still load-bearing for nearly every repo-scoped command.
