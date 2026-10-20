---
title: ti exec
summary: Run one command in a remote sandbox with a TiDB Cloud Filesystem workspace, live output, and exact remote exit status handling.
---

# ti exec

Runs one command in a remote sandbox with the selected TiDB Cloud Filesystem at `/workspace`. Stdout and stderr stream live, and the remote command's exit status is preserved. Runtime executions are never retried automatically. Only files under `/workspace` persist. Remote stdin and interactive terminals are not supported.

Use `ti update` or reinstall ti if the local capability check reports an incompatible Filesystem runtime.

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

Runtime flag parsing stops at `<command>`. Options after that boundary are passed unchanged to the remote command. Use `--` when the command starts with a dash. The exact bare `ti exec help` alias and `ti exec --help` show local help. `ti exec -w repo help` and `ti exec -- help` run a remote command named `help`.

## Options

- `-w, --workspace <string>`: Run in this Filesystem directory. Inside a matching local FUSE mount, the directory is inferred. Elsewhere, the default is the Filesystem root. Explicit `-w` does not drain local writes or create an isolation boundary.
- `-C, --cwd <string>`: Run in this subdirectory of the selected workspace.
- `-e, --env <string>`: Set `KEY=VALUE` remotely, or copy a local `KEY`. Repeatable.
- `-t, --timeout <duration>`: Stop the command after this duration. The server might use a shorter limit.
- `--fresh`: Start a new sandbox instead of reusing a warm sandbox. Persisted Filesystem data is kept.
- `--layer <string>`: Use a LayerFS layer ID, name, or `tag:key=value` reference.
- `--read-only`: Mount the workspace read-only.
- `-v, --verbose`: Show the sandbox, working directory, and duration.
- `--file-system-id <string>`: Select the Filesystem. You can also set `TI_FS_FILE_SYSTEM_ID`.
- `--fs-token <string>`: Set the Filesystem token. Prefer `TI_FS_TOKEN` so that the token does not enter process arguments. The Filesystem service enforces the token's permissions.
- `--help`: Display help information.
- `--version`: Display version information.

The global `--profile`, `--region`, and `--debug` options are accepted before the remote command. The structured-output options `--output` and `--query` are not supported because stdout and stderr stream unchanged. `--dry-run` is not supported.

## Examples

Select a Filesystem explicitly. Later examples assume `TI_FS_FILE_SYSTEM_ID` or `TI_FS_TOKEN` is set.

- Select a Filesystem and run in a remote directory:

    ```bash
    ti exec --file-system-id <file-system-id> -w repo npm test
    ```

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

The command returns the remote process's exact exit status. Runtime uses `124` for timeout, `130` for cancellation or deliberate detachment, and `255` for an unknown outcome. Remote commands can return those same statuses or ti error statuses such as `2`, `3`, `4`, and `5`; the status alone cannot identify its source. After an unknown outcome, inspect the workspace before running the command again.

The first Ctrl-C or SIGTERM requests cancellation and waits for workspace drain. A second interruption detaches, and the command might still be running.

## Related documentation

- [Run Commands in a TiDB Cloud Filesystem Sandbox](/ai/ti/reference/ti-runtime-exec.md)
- [TiDB Cloud CLI Command Reference](/ai/ti/reference/ti-cli-reference.md)
