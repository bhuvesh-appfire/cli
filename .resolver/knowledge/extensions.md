---
name: extensions
description: How gh extensions are discovered, installed, and dispatched - read before touching extension code
type: knowledge
scope:
  - pkg/cmd/extension/**
  - pkg/cmd/root/extension.go
  - pkg/cmd/root/root.go
updated: 2026-08-03 (IONE-959)
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - pkg/cmd/extension/manager.go
  - pkg/cmd/root/extension.go
  - pkg/cmd/factory/default.go
---

Extensions are third-party `gh <name>` subcommands that are *not* part of this codebase's `pkg/cmd/` tree.
They're either a git checkout of a repo following the `gh-<name>` naming convention, or a downloaded
prebuilt binary, installed under the extension manager's install directory
(`pkg/cmd/extension/manager.go:589` `installDir()`).

- `pkg/cmd/extension.Manager` (constructed once per process in `pkg/cmd/factory/default.go:272-289` as
  `extensionManager(f)`) owns discovery (`list`/`List`), install/upgrade/remove, and `Dispatch`.
- `pkg/cmd/root/extension.go:NewCmdExtension` wraps each discovered extension in a real `cobra.Command` with
  `DisableFlagParsing: true`, so all args/flags after the extension name pass through untouched to
  `Manager.Dispatch(args, ...)`, which execs the extension.
- Update checks happen out-of-band: `PreRun` kicks off `checkExtensionReleaseInfo` in a goroutine and
  `PostRun` non-blockingly drains the result channel, printing an upgrade nudge to stderr — it deliberately
  does not block command completion waiting for a network check (`pkg/cmd/root/extension.go:33-76`).
- `ExternalCommandExitError` wraps the extension binary's `*exec.ExitError` so the root command can
  propagate the extension's real exit code instead of gh's own error path.

## When modifying

- Extension-specific logic belongs in `pkg/cmd/extension/`, not in `pkg/cmd/root/` — `root.go` and
  `extension.go` are wiring only. This mirrors the "logic specific to a command should be kept within the
  command's package" rule in `docs/project-layout.md` for first-party commands.
- Do not modify `pkg/cmd/root/official_extension_stub.go` casually — it's generated/stub-related (see
  `pkg/cmd/root/official_extension_stub_test.go`) and covered by dedicated telemetry acceptance tests
  (`acceptance/testdata/telemetry/telemetry-for-official-extension-stub.txtar`,
  `no-telemetry-for-extension.txtar`) that assert extensions and official-extension-stubs are telemetered
  differently from built-in commands.
