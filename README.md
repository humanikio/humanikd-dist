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
installs third-party software (it prints the command) and never touches
credentials. Its last step offers to install the auto-start service for you.

Then check it and let the auto-start service run it **in the background**:

```bash
humanikd verify           # is the backend serveable?
humanikd service install  # run it as a background daemon (setup offers this too)
```

> `humanikd serve` also exists, but it runs in the **foreground** and blocks the
> terminal — it's for a quick test/debug, not how you run it day to day. The
> service (below) is the real daemon: no terminal, starts at login.

## Sign in to Claude once (agent devices)

If this machine runs the **Claude Code agent**, sign in a single time:

```bash
claude    # log in when prompted
```

That's all — you're set until the token expires and Claude asks you to sign in
again. Skip it and jobs fail with `authentication_failed`.

### Your connectors follow the Claude ACCOUNT, not the machine

Worth knowing before you rely on one. The agent inherits the MCP servers you
connected on this machine — Gmail, Drive, Linear, whatever you set up — so an
office running here can use them.

But **`claude.ai` connectors belong to the Claude account you are signed into.**
Sign into a different account (after hitting a usage limit, say) and every one of
them disappears from the agent. No configuration changes, nothing warns you, and
the device keeps reporting healthy with fewer abilities than it had yesterday.

```bash
humanikd verify    # lists your MCP servers and whether each is authenticated
```

Run that after any account change. Servers you registered on the device
yourself — `claude mcp add` + `claude mcp login`, or a stdio server — survive the
switch; the hosted `claude.ai` ones do not, and Google's (Gmail, Drive, Calendar)
can only be connected through claude.ai.

**Connecting a server is not the same as allowing it.** `allowed_tools` in
`~/.humanikd/config.yaml` decides what the agent may actually call; a tool that is
not listed is refused and its server never runs. `humanikd verify` warns when you
have servers connected but no `mcp__` tool allowed.

## Run it as a background daemon (recommended)

This is how you actually run humanikd — **no terminal to keep open.** `humanikd
setup` offers it at the end; you can also do it directly. `install` picks the
right kind for your device:

```bash
# macOS agent device — runs as YOU (reads your keychain), no sudo:
humanikd service install
# local-model / Linux / Windows — system service:
sudo humanikd service install

humanikd service status
```

It runs in the background, starts at login, and restarts after the machine wakes
or the process exits.

## Two roles — pick one per machine

| | Local model | Claude Code agent |
|---|---|---|
| Serves | completions from Ollama (or compatible) | an agent turn from the Claude Code CLI |
| Needs | `ollama serve` running | `claude` installed **and signed in** |
| Config | `backend.ollama` | `backend.claude_agent.enabled: true` |
| Auto-start | `sudo humanikd service install` | `humanikd service install` (no sudo — LaunchAgent) |

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
| `humanikd service <install\|start\|stop\|status>` | **Run as a background daemon** (recommended; also on boot) |
| `humanikd serve` | Run in the **foreground** — a quick test; blocks the terminal |
| `humanikd verify` | Check the backend is serveable (+ service state) |
| `humanikd status` | Config, backend, enrollment, **version** |
| `humanikd version` | Print the installed version |
| `humanikd upgrade` | Check for a newer release |

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
