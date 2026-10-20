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

- Install or update the TiDB Cloud CLI. Its installer manages the required Filesystem runtime.
- Select a Filesystem with `--file-system-id` or `TI_FS_FILE_SYSTEM_ID`, or supply `--fs-token` or `TI_FS_TOKEN` to derive its ID. A profile stores credentials and placement; it does not select a default Filesystem. Runtime execution requires a Filesystem API-key credential; a delegated Vault token cannot run commands.
- Make sure that Runtime is enabled for the selected Filesystem. Otherwise, the command reports that remote execution is not enabled.

## Run a command

Select a Filesystem and use `-w` to run in one of its directories:

```bash
ti exec --file-system-id <file-system-id> -w repo npm test
```

The following examples assume `TI_FS_FILE_SYSTEM_ID` or `TI_FS_TOKEN` is set. To run a shell command line, pass it as a single argument:

```bash
ti exec -w repo 'npm ci && npm test'
```

Inside a matching local FUSE mount, omit `-w` to drain pending local writes and run in the corresponding remote directory:

```bash
cd ~/workspace/repo/web
ti exec npm test
```

Use `-e` or `--env` to set a remote environment variable. A name without a value copies the variable from your local environment:

```bash
ti exec -e CI=1 -e GITHUB_TOKEN -t 20m -- make -j8 test
```

Runtime flag parsing stops at the remote command. Use `--` to make the boundary explicit or when the command itself starts with a dash. `ti exec help` and `ti exec --help` show local help; `ti exec -w repo help` and `ti exec -- help` run a remote command named `help`. A single argument that contains spaces or shell syntax runs through `/bin/sh -c`; multiple arguments run directly as the specified argument vector.

## Choose the workspace and sandbox

The complete Filesystem is available at `/workspace`. `-w` selects the working directory within it, and `-C` or `--cwd` selects a subdirectory of that directory. These options do not create an isolation boundary.

- Inside a matching local FUSE mount, the workspace and current directory are inferred automatically. WebDAV mounts are not inferred, so pass `-w` when you use one.
- Outside a mount, the default workspace is the Filesystem root. An explicit `-w` does not drain local writes.
- Files under `/workspace` remain in the Filesystem after the command exits. Files elsewhere in the sandbox are disposable.
- Consecutive commands can reuse a warm sandbox. Use `--fresh` when you need a new sandbox; persisted Filesystem data is kept.
- Use `--layer <reference>` to run against a LayerFS view. Use `--read-only` when the command must not change workspace files.

The remote process runs as a normal user named `agent` with `HOME=/home/agent`. It cannot become root.

## Handle output, exit status, and cancellation

The command's stdout and stderr stream live and remain separate. Remote stdin and interactive terminals are not supported. The TiDB Cloud CLI does not convert Runtime output to JSON or text tables, so `--output`, `--query`, and `--dry-run` are not supported by `ti exec`.

When the remote command exits, `ti exec` exits with the same status. Other Runtime outcomes use the following statuses:

| Status | Meaning |
| --- | --- |
| `124` | The command reached the `--timeout` limit and was stopped. |
| `126` or `127` | The command is not executable or was not found. |
| `130` | Cancellation or deliberate detachment. |
| `255` | The command outcome could not be confirmed. |

Remote commands can return any status, including `124`, `130`, `255`, and ti error statuses such as `2`, `3`, `4`, and `5`. The status alone cannot identify its source.

The first Ctrl-C or SIGTERM requests cancellation and waits for workspace changes to drain. A second interruption detaches immediately, and the command might keep running.

Runtime executions are never retried automatically. After exit status `255`, inspect the workspace before deciding whether to run the command again because the previous command might already have run.

## Protect credentials

Prefer environment variables or a configured profile instead of putting a Filesystem token on the command line. The TiDB Cloud CLI passes the selected token only to the Filesystem runtime; the token is not added to the remote command environment.

When you use `-e NAME` to copy a local variable, the value is intentionally sent to the remote command. Use a secret manager and grant only the variables that the command needs.

## Related documentation

- [`ti exec` command reference](/ai/ti/reference/commands/ti/ti-exec.md)
- [TiDB Cloud CLI Configuration and Credentials](/ai/ti/reference/ti-configuration-and-credentials.md)
- [TiDB Cloud Filesystem CLI Command Reference](/ai/ti/reference/ti-filesystem.md)
