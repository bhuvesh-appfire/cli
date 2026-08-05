---
name: architecture
description: Command dispatch flow, factory wiring, and the precedence rules between core commands, extensions, aliases, and BaseRepo resolution
type: knowledge
scope: global
updated: '2026-08-05'
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - pkg/cmd/root/root.go
  - pkg/cmd/factory/default.go
  - pkg/cmdutil/factory.go
---

```mermaid
flowchart TD
    Main["cmd/gh/main.go"] --> GhcmdMain["internal/ghcmd.Main()"]
    GhcmdMain --> FactoryNew["factory.New() builds cmdutil.Factory\n(HttpClient, GitClient, Remotes, Prompter, ExtensionManager)"]
    FactoryNew --> RootCmd["root.NewCmdRoot(f, telemetry, ...)"]
    RootCmd --> Core["Core commands added first\n(issue, pr, repo, api, ...)"]
    RootCmd --> HelpTopics["Help topics (root/help_topic.go)"]
    RootCmd --> Extensions["Installed extensions\n(skipped if name collides with a core command)"]
    RootCmd --> Aliases["User aliases from config\n(skipped if invalid name/expansion)"]
    RootCmd --> Stubs["Official extension stubs\n(skipped if name already taken by core/extension/alias)"]
    Core --> Dispatch["cobra finds matching cmd.RunE"]
    Extensions --> Dispatch
    Aliases --> Dispatch
    Stubs --> Dispatch
```

## Command name resolution precedence

Registration order in `NewCmdRoot` (`pkg/cmd/root/root.go`) is: core commands -> help topics -> installed extensions -> user aliases -> official extension stubs. Each later category explicitly does `cmd.Find([]string{name})` first and skips itself if a command of that name already exists. So a locally installed extension named e.g. `gh-issue` would never shadow the built-in `gh issue`, and a stub for an official extension never shows up if the user already has an alias or a real extension with that name.

## Two BaseRepo resolution strategies

`cmdutil.Factory.BaseRepo` is set twice, differently, in `root.go`:

- The plain `factory.BaseRepoFunc` (used for `NewCmdRoot`'s initial factory) just takes `Remotes()[0]` - no API calls.
- `factory.SmartBaseRepoFunc(f)` is substituted for a **copy** of the factory (`repoResolvingCmdFactory := *f`) used only by repo-context commands (`pr`, `issue`, `repo`, `release`, `run`, `workflow`, `label`, `cache`, `api`, `browse`, `discussion`, `org`, `ruleset`, `agent-task`). It hits the GraphQL API to resolve repo networks/parents and can prompt the user to run `gh repo set-default` when remotes are ambiguous.

Remotes themselves are ordered `upstream` > `github` > `origin` > others (see the extensive doc comment on `BaseRepoFunc` in `pkg/cmd/factory/default.go`), and only remotes for hosts the user is authenticated against are retained. If you add a new command that needs a base repo, wire it through `repoResolvingCmdFactory`, not the raw `f`, or you'll silently get the "first remote, no API" behavior instead of smart resolution.

## Telemetry recording

`cmdutil.RecordTelemetryForSubcommands` (called once, on the fully-assembled root command) walks the whole command tree and wraps every leaf `RunE` to emit a `command_invocation` event with sorted flag names, unless `cmdutil.DisableTelemetry`/`DisableTelemetryForSubcommands` set the `telemetry: disabled` annotation on that command first. `cmdutil.DisableAuthCheck` works the same way via a different annotation. When adding a new command that shouldn't be tracked or shouldn't require auth, set the annotation before `RecordTelemetryForSubcommands`/the auth-check runs, i.e. before `NewCmdRoot` returns.
