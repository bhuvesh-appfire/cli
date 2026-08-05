---
name: gotchas
description: Non-obvious runtime behaviors around host detection and agent-task auth gating that aren't documented elsewhere
type: knowledge
scope: global
updated: '2026-08-05'
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - internal/ghinstance/host.go
  - pkg/cmd/agent-task/agent_task.go
---

## `ghinstance` host special-casing

`internal/ghinstance/host.go` builds API/Gist/REST URLs from a hostname and treats four cases differently, in this priority order: `garage.github.com` (an internal GitHub host, checked via unexported `isGarage`) and enterprise hosts (`ghauth.IsEnterprise`) both get `/api/v3/` and `/api/graphql` REST/GraphQL prefixes; `github.localhost` gets `http://` (not `https://`) and an `api.` subdomain for dev/test environments; everything else gets the normal `api.github.com`-style prefix. `CategorizeHost` (used for telemetry/logging) buckets hosts into `"github.com"`, `"ghes"`, `"tenancy"`, or `"uncategorized"` - note `garage.github.com` falls into `"uncategorized"` there even though URL construction special-cases it, since `CategorizeHost` doesn't call `isGarage`. If you touch host categorization, keep that asymmetry in mind or fix both call sites together.

## `gh agent-task` requires a device-flow OAuth token

`pkg/cmd/agent-task/agent_task.go`'s `requireOAuthToken` gates every `agent-task` subcommand via `PersistentPreRunE`. It rejects the command unless the active token both (a) came from source `"oauth_token"` or `"keyring"` (not `env`/`GH_TOKEN`/etc.) and (b) has the `gho_` prefix. A PAT (`ghp_`) or fine-grained token, even if valid and scoped correctly, is rejected with "this command requires an OAuth token. Re-authenticate with: gh auth login". Also hard-blocked entirely on enterprise hosts. Any new agent-task subcommand added under `pkg/cmd/agent-task/` inherits this check automatically since it's on the parent command.
