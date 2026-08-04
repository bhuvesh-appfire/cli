---
name: architecture
description: Verified call-graph from process entry point through factory construction to command dispatch
type: knowledge
scope: global
updated: '2026-08-04'
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - cmd/gh/main.go
  - internal/ghcmd/cmd.go
  - pkg/cmd/factory/default.go
  - pkg/cmd/root/root.go
  - api/client.go
  - git/client.go
---

```mermaid
flowchart TD
    main["cmd/gh/main.go: main()"] --> ghcmdMain["internal/ghcmd.Main()"]
    ghcmdMain --> config["internal/config.NewConfig()"]
    ghcmdMain --> telemetry["internal/telemetry / ghtelemetry.Service"]
    ghcmdMain --> factoryNew["pkg/cmd/factory.New()"]
    factoryNew --> httpClient["api.NewClientFromHTTP via HttpClientFunc"]
    factoryNew --> gitClient["git.Client (newGitClient)"]
    factoryNew --> remotes["context.Remotes (remotesFunc, depends on GitClient+Config)"]
    factoryNew --> baseRepo["BaseRepoFunc (depends on Remotes)"]
    factoryNew --> prompter["internal/prompter (depends on Config+IOStreams)"]
    factoryNew --> browser["internal/browser (depends on Config+IOStreams)"]
    factoryNew --> extMgr["pkg/cmd/extension.Manager (depends on Config+HttpClient+IOStreams)"]
    ghcmdMain --> rootNew["pkg/cmd/root.NewCmdRoot(factory, ...)"]
    rootNew --> cmdTree["pkg/cmd/<command>/<subcommand> packages"]
    cmdTree --> apiPkg["api/ (GraphQL + REST via Client.GraphQL/Client.REST)"]
    cmdTree --> gitPkg["git/ (local repo introspection)"]
    cmdTree --> cmdutilPkg["pkg/cmdutil (Factory, error types, flag helpers)"]
    cmdTree --> iostreamsPkg["pkg/iostreams (TTY/color/pager)"]
    rootNew --> extension["pkg/cmd/root/extension.go (dispatches to gh extensions as external commands)"]
```

Notes on non-obvious edges:
- `factory.New()` wires dependencies in a strict order documented at `pkg/cmd/factory/default.go:38-44` - `GitClient` before `Remotes` before `BaseRepo` before `Prompter`/`Browser`/`ExtensionManager`/`Branch`. Each depends on the previous, so reordering breaks construction.
- `BaseRepoFunc` (`pkg/cmd/factory/default.go:74`) always picks remotes[0] from an already-sorted list (`upstream` > `github` > `origin` > others); it does not itself do the sorting, `remotesFunc` does.
- Extension commands are distinguished from built-in ones purely by `cobra.Command.GroupID == "extension"` (`internal/ghcmd/cmd.go:277-278`), which is how `gh help <ext>` gets rewritten to `<ext> --help`.
- Telemetry service selection in `Main()` branches on `cfg.Telemetry().Value` (Disabled/Logged/Enabled) and independently forces no-op if `mightBeGHESUser(cfg)` is true - GHES users never get real telemetry even if the config says enabled.
