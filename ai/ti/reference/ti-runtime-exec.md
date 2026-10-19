---
title: Run Commands in a TiDB Cloud Filesystem Sandbox
summary: Run commands in a remote sandbox with a TiDB Cloud Filesystem workspace, live output, durable files, and exact exit status handling.
---

# Run Commands in a TiDB Cloud Filesystem Sandbox

`ti exec` runs one command in a remote sandbox with your selected TiDB Cloud Filesystem mounted at `/workspace`. The command sees durable workspace files without requiring you to create, mount, or delete a sandbox.

> **Note:**
>
> The TiDB Cloud Command Line Interface — `ti` — is currently in preview. Its features and command-line interface might change without prior notice.

## Prerequisites

- Install the TiDB Cloud CLI and its matching Filesystem runtime. The installer and updater reject a runtime that does not expose the `drive9 exec` command. A ti release that includes `ti exec` is published only after the matching Drive9 artifact passes the live Runtime release test.
- Select a Filesystem through a configured profile, `TI_FS_TOKEN`, or the `--file-system-id` and `--fs-token` options. Runtime execution requires a Filesystem API-key credential; a delegated Vault token cannot run commands.
- Make sure that Runtime is enabled for the selected Filesystem. Otherwise, the command reports that remote execution is not enabled.

## Run a command

When you are inside a local FUSE mount, `ti exec` uploads pending changes and runs in the matching remote directory:

```bash
cd ~/workspace/repo/web
ti exec npm test
```

From another directory, use `-w` or `--workspace` to select a directory in the Filesystem:

```bash
ti exec -w repo 'npm ci && npm test'
```

Use `-e` or `--env` to set a remote environment variable. A name without a value copies the variable from your local environment:

```bash
ti exec -e CI=1 -e GITHUB_TOKEN -t 20m -- make -j8 test
```

Runtime flag parsing stops at the remote command. Use `--` to make the boundary explicit or when the command itself starts with a dash. A single argument that contains spaces or shell syntax runs through `/bin/sh -c`; multiple arguments run directly as the specified argument vector.

## Choose the workspace and sandbox

The complete Filesystem is available at `/workspace`. The selected workspace directory becomes the command's workspace root, and `-C` or `--cwd` selects a subdirectory within it.

- Inside a matching local FUSE mount, the workspace and current directory are inferred automatically. WebDAV mounts are not inferred, so pass `-w` when you use one.
- Outside a mount, the default workspace is the Filesystem root.
- Files under `/workspace` remain in the Filesystem after the command exits. Files elsewhere in the sandbox are disposable.
- Consecutive commands can reuse a warm sandbox. Use `--fresh` when you need a new sandbox.
- Use `--layer <reference>` to run against a LayerFS view. Use `--read-only` when the command must not change workspace files.

The remote process runs as a normal user named `agent` with `HOME=/home/agent`. It cannot become root.

## Handle output, exit status, and cancellation

The command's stdout and stderr stream live and remain separate. The TiDB Cloud CLI does not convert Runtime output to JSON or text tables, so `--output`, `--query`, and `--dry-run` are not supported by `ti exec`.

When the remote command exits, `ti exec` exits with the same status. Other Runtime outcomes use the following statuses:

| Status | Meaning |
| --- | --- |
| `124` | The command reached the `--timeout` limit and was stopped. |
| `126` or `127` | The command is not executable or was not found. |
| `130` | The command was canceled with Ctrl-C. |
| `255` | The connection was lost and the command outcome is unknown. |

The first Ctrl-C requests cancellation and waits for workspace changes to drain. A second Ctrl-C detaches immediately, and the command might keep running.

Runtime executions are never retried automatically. After exit status `255`, inspect the workspace before deciding whether to run the command again because the previous command might already have run.

## Protect credentials

Prefer environment variables or a configured profile instead of putting a Filesystem token on the command line. The TiDB Cloud CLI passes the selected token only to the Filesystem runtime; the token is not added to the remote command environment.

When you use `-e NAME` to copy a local variable, the value is intentionally sent to the remote command. Use a secret manager and grant only the variables that the command needs.

## Related documentation

- [`ti exec` command reference](/ai/ti/reference/commands/ti/ti-exec.md)
- [TiDB Cloud CLI Configuration and Credentials](/ai/ti/reference/ti-configuration-and-credentials.md)
- [TiDB Cloud Filesystem CLI Command Reference](/ai/ti/reference/ti-filesystem.md)
