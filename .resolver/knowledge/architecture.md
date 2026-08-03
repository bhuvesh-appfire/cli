---
name: architecture
description: Real request/execution flow from binary entrypoint through to the GitHub API and telemetry, as verified in code
type: knowledge
scope: global
updated: 2026-08-03 (IONE-959)
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - cmd/gh/main.go
  - internal/ghcmd/cmd.go
  - pkg/cmd/root/root.go
  - pkg/cmd/factory/default.go
  - api/client.go
  - api/http_client.go
  - internal/config/config.go
  - internal/telemetry/telemetry.go
  - internal/gh/ghtelemetry/telemetry.go
  - internal/featuredetection/feature_detection.go
  - pkg/extensions/extension.go
---

`AGENTS.md`'s "Architecture" section already gives the entrypoint one-liner (`cmd/gh/main.go` → `internal/ghcmd.Main()` → `pkg/cmd/root.NewCmdRoot()`) and the key-packages list. This diagram fills in what's actually wired together at runtime and where each `pkg/cmd/<command>` talks to the outside world.

```mermaid
flowchart TD
    Bin["cmd/gh/main.go"] --> Main["internal/ghcmd.Main()"]
    Main --> Config["internal/config.NewConfig()"]
    Main --> Detect["internal/agents.Detect() (agent env-var sniff)"]
    Main --> Telemetry["internal/telemetry.Service"]
    Main --> Factory["pkg/cmd/factory.New() -> cmdutil.Factory"]
    Factory --> Root["pkg/cmd/root.NewCmdRoot()"]
    Root --> Cmds["pkg/cmd/<command>/<subcommand>"]

    Factory -. provides .-> HTTP["Factory.HttpClient (auth headers attached)"]
    Factory -. provides .-> PlainHTTP["Factory.PlainHttpClient (no auth headers)"]
    Factory -. provides .-> ExtHTTP["Factory.ExternalHttpClient (non-GitHub hosts)"]
    Factory -. provides .-> GitClient["git.Client (shells out to local git)"]
    Factory -. provides .-> ExtMgr["pkg/extensions.ExtensionManager"]

    Cmds --> APIClient["api.Client (GraphQL + REST wrapper)"]
    APIClient --> HTTP
    APIClient -->|"GraphQL/REST"| GitHubAPI["GitHub API (github.com or GHES)"]

    Cmds -.->|"feature-gated behavior"| FeatureDetect["internal/featuredetection.Detector"]
    FeatureDetect -->|"queries schema/version"| GitHubAPI

    Cmds --> GitClient
    GitClient -->|"exec"| LocalGit["local .git repo"]

    Telemetry -->|"batched events"| TelemetryBackend["telemetry collector (unless GHES/disabled)"]

    Cmds -. loads/saves .-> ConfigFile["~/.config/gh/ (hosts.yml, config.yml)"]
    Config --> ConfigFile
```

Non-obvious edges:
- `Factory.HttpClient`, `PlainHttpClient`, and `ExternalHttpClient` are three separate clients, not layers of one — see [[gotchas]] for when to use each.
- `internal/featuredetection` queries the target host's GraphQL schema/version to decide GHES-vs-github.com behavior at runtime; it's not a compile-time or config-time switch.
- Telemetry is force-disabled whenever the user looks like a GHES user (enterprise token env vars or an enterprise host in config) — see `mightBeGHESUser` in `internal/ghcmd/cmd.go`.
