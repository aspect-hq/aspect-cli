---
name: aspect-cli
description: Work with Aspect, the media asset management platform, through its `aspect` CLI — upload and download media, mount projects as local drives, and track background transfers. Use when the user mentions Aspect, an aspect:// path, an app.aspect.inc link, or moving files to or from Aspect.
---

# Aspect CLI

`aspect` moves files between the local machine and Aspect and mounts Aspect projects as local volumes. Remote locations are written as `aspect://` URLs.

## Scope

The CLI covers where Aspect meets the local machine: file transfers, mounted drives, and the local background daemon that powers them. For working within Aspect — asset searching, asset understanding, collections, metadata, comments, sharing, user management — use the Aspect MCP server's tools instead, and use the CLI only to move bytes. If the MCP server is not connected yet, connect it: `https://api.aspect.inc/mcp` (Streamable HTTP; OAuth sign-in or an `sk_` API key as a Bearer token; setup guide at https://aspect.inc/docs/api-reference/mcp).

## Setup

Check for the CLI with `aspect --version`. If missing, install it:

- macOS/Linux: `curl -fsSL https://aspect.inc/cli/install.sh | sh`
- Windows (PowerShell): `powershell -ExecutionPolicy Bypass -c "irm https://aspect.inc/cli/install.ps1 | iex"`

The CLI self-updates with `aspect upgrade`.

## Authentication

Set the `ASPECT_API_KEY` environment variable (keys start with `sk_`). It authenticates every command directly — no login, no stored state — and overrides any stored credential. Never pass API keys as command arguments, and never print them.

Users create API keys in Aspect settings. A user who logged in interactively (`aspect auth login`) already has a stored credential, so commands may work with no env var at all. Check what is in effect with `aspect auth status --json`. Do not run `aspect auth login` non-interactively — it errors without `--token`; use the env var instead.

## Rules

- Always pass `--json`. It is a global flag on every command, and it guarantees the command never prompts. `aspect --help --json` returns a machine-readable list of all commands; `aspect <command> --help --json` returns that command's flags, actions, and examples.
- stdout carries the JSON result and nothing else. Progress output goes to stderr, and only when stderr is a TTY — never try to parse it.
- Exit codes: `0` success, `1` command failure, `2` usage error. Errors emit `{"error":{"code":"...","message":"..."}}` on stderr with stable, branchable codes such as `usage`, `not_logged_in`, `invalid_api_key`, `api_request_failed`, `network_timeout`, `network_unreachable`, `not_found`, `ambiguous_name`, `not_a_directory`, `source_not_found`, `directory_conflict`, `destination_not_writable`, `daemon_error`. Some commands exit `1` with a normal JSON result on stdout rather than an error envelope — `doctor` with failing checks, `transfer status`/`wait` on a failed transfer, a transfer with per-file failures — so on exit `1`, check stdout for a result before assuming stderr holds an envelope.

## Addressing

Remote paths are `aspect://<workspace>/<project>/<path>`, for example `aspect://Acme/MyProject/Renders`. Prefer naming the workspace explicitly. `~` stands for the user's default workspace (`aspect://~/MyProject/Renders`) — use it only when the user has one set and means it.

There is no `--workspace` flag. `~` resolves through the `ASPECT_WORKSPACE` env var (name or id), then the user's stored default, then sole workspace membership. Never set the stored default (`aspect workspace use`) unless the user explicitly asks for that; when the default is missing or wrong, name the workspace in the URL instead. List workspaces with `aspect workspace list --json`.

## Transfers

```sh
aspect upload <local-file-or-dir>... aspect://<workspace>/<project>/<path> [--replace | --keep-both] [--detach] --json
aspect download aspect://<workspace>/<project>/<path> [local-dir] [--variant original|stream_proxy|preview] [--replace | --keep-both] [--detach] --json
```

Name conflicts are skipped by default; `--replace` overwrites, `--keep-both` keeps both copies (mutually exclusive).

`--variant` picks what to download: `original` (default), `stream_proxy` (the H.264 MP4 proxy, up to 1080p, with AAC audio when the source has audio, saved as `<name>_proxy.mp4`), or `preview` (the poster image, saved as `<name>_preview.<ext>`). Prefer `stream_proxy` when the bytes are for viewing or analysis rather than editing — it is far smaller than most originals. In a directory download, assets that have no such variant yet are left out rather than failing it; a single asset that lacks it fails the download.

For large or long-running transfers, add `--detach`: the command returns immediately with a transfer id. Then either block on completion with `aspect transfer wait <id> --json` or poll `aspect transfer status <id> --json`. `aspect transfer list --json` shows all detached transfers; `aspect transfer cancel <id>` stops one.

## Mounts and the daemon

`aspect mount aspect://Acme/MyProject --json` mounts a project (or a directory under it) as a local volume, so its files can be read and written with ordinary filesystem tools; `aspect unmount` removes it. Mounting needs platform prerequisites (FUSE on Linux, WinFSP on Windows); non-interactive runs never auto-install them — a structured failure explains what is missing, and `aspect doctor --json` diagnoses the environment.

`aspect pin add aspect://Acme/MyProject/Renders --json` keeps a project, directory, or file downloaded for offline use on mounts (re-pinning is a no-op that returns the same pin); `aspect pin list --json` shows every pin with its sync status and synced/total bytes, and `aspect pin remove <url | pin-id>` unpins. A file that is offline only through a pinned parent has no pin of its own (`not_pinned`) — unpin the parent.

`aspect settings list --json` lists every setting for Aspect on this computer (today, the cache and pin storage limits) with its name, current `value` and `unit`, and what it controls — read the names from there rather than guessing; `aspect settings set <setting> <value>` changes one, e.g. `aspect settings set cache-limit 200GB`. Sizes need a unit. The service refuses a limit under its minimum or a pin limit below what is already pinned (`daemon_error`).

The background daemon starts automatically whenever a command needs it — never start it yourself; the `aspect daemon` commands exist for troubleshooting. `aspect status --json` shows the daemon, active mounts, an overall pin sync summary, and transfers at a glance.