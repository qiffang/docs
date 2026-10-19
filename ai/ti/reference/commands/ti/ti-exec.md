---
title: ti exec
summary: Run one command in a remote sandbox with a TiDB Cloud Filesystem workspace, live output, and exact remote exit status handling.
---

# ti exec

Runs one command in a remote sandbox with the selected TiDB Cloud Filesystem at `/workspace`. Stdout and stderr stream live, and the remote command's exit status is preserved. Runtime executions are never retried automatically.

The installed `ti-drive9` Filesystem runtime must support the `drive9 exec` command. The installer and updater reject older runtime binaries before installation.

> **Note:**
>
> The TiDB Cloud Command Line Interface — `ti` — is currently in preview. Its features and command-line interface might change without prior notice.

## Syntax

```text
ti exec
  [-w <string>]
  [-C <string>]
  [-e <string>]...
  [-t <duration>]
  [--fresh]
  [--layer <string>]
  [--read-only]
  [-v]
  [--file-system-id <string>]
  [--fs-token <string>]
  [--help]
  [--version]
  [--]
  <command> [argument]...
```

Runtime flag parsing stops at `<command>`. Options after that boundary are passed unchanged to the remote command. Use `--` when the command starts with a dash.

## Options

- `-w, --workspace <string>`: Run in this Filesystem directory. Inside a matching local FUSE mount, the directory is inferred. Elsewhere, the default is the Filesystem root.
- `-C, --cwd <string>`: Run in this subdirectory of the selected workspace.
- `-e, --env <string>`: Set `KEY=VALUE` remotely, or copy a local `KEY`. Repeatable.
- `-t, --timeout <duration>`: Stop the command after this duration. The server might use a shorter limit.
- `--fresh`: Start a new sandbox instead of reusing a warm sandbox.
- `--layer <string>`: Use a LayerFS layer ID, name, or `tag:key=value` reference.
- `--read-only`: Mount the workspace read-only.
- `-v, --verbose`: Show the sandbox, working directory, and duration.
- `--file-system-id <string>`: Select the Filesystem. You can also set `TI_FS_FILE_SYSTEM_ID`.
- `--fs-token <string>`: Set the Filesystem token. Prefer `TI_FS_TOKEN` so that the token does not enter process arguments. The Filesystem service enforces the token's permissions.
- `--help`: Display help information.
- `--version`: Display version information.

The global `--profile`, `--region`, and `--debug` options are accepted before the remote command. The structured-output options `--output` and `--query` are not supported because stdout and stderr stream unchanged. `--dry-run` is not supported.

## Examples

- Run in the directory that matches the current FUSE mount path:

    ```bash
    cd ~/workspace/repo/web
    ti exec npm test
    ```

- Select a remote workspace and run a shell command line:

    ```bash
    ti exec -w repo 'npm ci && npm test'
    ```

- Set environment variables and a timeout:

    ```bash
    ti exec -e CI=1 -e GITHUB_TOKEN -t 20m -- make -j8 test
    ```

- Start a fresh, read-only sandbox on a layer:

    ```bash
    ti exec --fresh --layer experiment --read-only -- make test
    ```

## Exit status

The command returns the remote process's exit status. Status `124` means timeout, `130` means cancellation, and `255` means that the command outcome is unknown. For status `255`, inspect the workspace before running the command again.

## Related documentation

- [Run Commands in a TiDB Cloud Filesystem Sandbox](/ai/ti/reference/ti-runtime-exec.md)
- [TiDB Cloud CLI Command Reference](/ai/ti/reference/ti-cli-reference.md)
