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
