# humanikd

The downloadable **device daemon** for HumanikOS. It runs on a machine you own,
dials out to the HumanikOS gateway, and serves work locally — a local model
(Ollama) or a local Claude Code agent. Billed as `byok` ($0 platform credits):
you bring the compute.

The **private key never leaves your machine.** The daemon generates an Ed25519
keypair at enrollment and sends only the public half.

> This repository holds the **released binaries + installer only.** The source is
> maintained privately; these are the signed, runnable artifacts.

## Install

```bash
curl -fsSL https://github.com/humanikio/humanikd-dist/releases/latest/download/install.sh | sh
humanikd setup   # guided first-run — START HERE
```

`setup` walks the steps in order and stops at the first real blocker. It never
installs anything itself (it prints the command) and never touches credentials.

Then:

```bash
humanikd verify        # is the backend serveable?
humanikd serve         # run it
```

## Two roles — pick one per machine

| | Local model | Claude Code agent |
|---|---|---|
| Serves | completions from Ollama (or compatible) | an agent turn from the Claude Code CLI |
| Needs | `ollama serve` running | `claude` installed **and signed in** |
| Config | `backend.ollama` | `backend.claude_agent.enabled: true` |

## Verify a download (recommended)

Every binary is checksummed and **cosign-signed** (keyless, Sigstore):

```bash
sha256sum -c humanikd-<os>-<arch>.sha256
cosign verify-blob --signature humanikd-<os>-<arch>.sig humanikd-<os>-<arch>
```

## Manual download

If you'd rather not pipe to a shell, grab the binary for your platform from the
[latest release](../../releases/latest), `chmod +x` it, and put it on your `PATH`.

Targets: `humanikd-darwin-arm64`, `humanikd-darwin-amd64`, `humanikd-linux-amd64`,
`humanikd-linux-arm64`, `humanikd-windows-amd64.exe`.

## Config & commands

`humanikd setup` writes `~/.humanikd/config.yaml` for you. Common commands:

| Command | Does |
|---|---|
| `humanikd setup` | Guided first run |
| `humanikd enroll --code <CODE>` | Pair this machine (get the code in the console) |
| `humanikd serve` | Run the daemon |
| `humanikd verify` | Check the backend is serveable |
| `humanikd status` | Config, backend, enrollment, **version** |
| `humanikd version` | Print the installed version |
| `humanikd upgrade` | Check for a newer release |
| `humanikd service <install\|start\|stop\|status>` | Run on boot (native service) |

## Staying up to date

The installer always fetches the **latest** release, so re-running it upgrades in
place:

```bash
curl -fsSL https://github.com/humanikio/humanikd-dist/releases/latest/download/install.sh | sh
```

`humanikd upgrade` tells you whether a newer version exists (it doesn't modify
anything — it just checks and prints the command above). To pin a specific version
instead of latest:

```bash
HUMANIKD_VERSION=v0.1.0 \
  curl -fsSL https://github.com/humanikio/humanikd-dist/releases/latest/download/install.sh | sh
```
