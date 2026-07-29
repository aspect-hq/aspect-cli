# Aspect CLI

The command-line interface for Aspect, the media asset management platform — file transfers, mounting projects as local drives, and daemon control.

## Install

macOS/Linux:

```sh
curl -fsSL https://aspect.inc/cli/install.sh | sh
```

Windows (PowerShell):

```powershell
powershell -ExecutionPolicy Bypass -c "irm https://aspect.inc/cli/install.ps1 | iex"
```

These installers work once the first release is published. After installing, the CLI keeps itself up to date with `aspect upgrade`.

Prebuilt binaries are attached to each [Release](https://github.com/aspect-hq/aspect-cli/releases) per platform: darwin-arm64, darwin-amd64, linux-amd64, linux-arm64, windows-amd64.

## Using with AI agents

The CLI is built for agents: every command supports a global `--json` flag (including `aspect --help --json` for a machine-readable command catalog), `--json` guarantees no interactive prompts, errors carry stable codes, and `ASPECT_API_KEY` authenticates without a stored login.

An [agent skill](skills/aspect-cli/SKILL.md) teaches agents (Claude Code, Cursor, Codex, and others) how to use the CLI. Install it with:

```sh
npx skills add aspect-hq/aspect-cli
```
