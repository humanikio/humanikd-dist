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

**macOS / Linux**

```bash
curl -fsSL https://github.com/humanikio/humanikd-dist/releases/latest/download/install.sh | sh
humanikd setup   # guided first-run — START HERE
```

**Windows** — the line above will not work here. PowerShell aliases `curl` to
`Invoke-WebRequest`, which rejects `-fsSL` before anything is downloaded, and
`install.sh` supports macOS and Linux only. Download the binary and put it on
your PATH:

```powershell
$dir = "$env:LOCALAPPDATA\Programs\humanikd"
New-Item -ItemType Directory -Force -Path $dir | Out-Null
Invoke-WebRequest -Uri "https://github.com/humanikio/humanikd-dist/releases/latest/download/humanikd-windows-amd64.exe" -OutFile "$dir\humanikd.exe"
[Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path","User") + ";$dir", "User")
```

**Then open a NEW PowerShell window** before continuing — Windows reads `Path`
only when a shell starts, so the window you just ran that in still cannot see
`humanikd`. In the new window:

```powershell
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
# Agent device (macOS / Windows) — runs as YOU, no sudo or admin:
humanikd service install
# Local-model device — system service:
sudo humanikd service install

humanikd service status
```

It runs in the background, starts when you log in, and restarts after the machine
wakes or the process exits.

**Why agent devices run as you, not as a system service.** Claude Code's
credentials belong to your account — the macOS login keychain, or your Windows
user profile — and on Windows your device's enrollment key is encrypted against
your account as well. A service running as root or LocalSystem cannot read any of
it, so it would start and immediately exit. On macOS that means a LaunchAgent; on
Windows, a logon task. Both install with the plain command above.

The consequence is the same on both: **it runs while you are logged in.** Logging
out stops it. There is no version that runs logged-out without storing your
password, and it would have nothing to work with if it did.

> **Windows, upgrading from v0.1.9 or earlier:** those builds registered a system
> service that could never start. Remove it once, from an elevated prompt:
> `sc.exe delete humanikd` — then `humanikd service install` as normal.

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

## What this writes to your disk

Everything lives under `~/.humanikd/`. Nothing is written outside it.

| Path | What |
|---|---|
| `identity.json` | This machine's enrollment key — **back it up**, it cannot be reissued |
| `config.yaml` | Your settings |
| `ws/<tenant>/<workspace>/` | Working directory for agent runs, one per workspace |

### Files a job brings with it

> **Planned, not in the current release.** Listed here so it is not a surprise
> when it lands.

Some turns arrive with a file rather than only text — a screenshot the agent is
looking at, a PDF, a spreadsheet. Because Claude Code takes text on its command
line and nothing else, the daemon writes those files into the run's own workspace
folder and tells the agent where they are:

```
~/.humanikd/ws/<tenant>/<workspace>/.attachments/<job>/frame.png
```

**You never have to clean this up.** The daemon does it:

- anything untouched for **2 hours** is removed, checked every 15 minutes
- during a long conversation, only the **last 3 turns** of files are kept
- **on every start, the folder is emptied** — nothing can be in use then, so this
  also clears anything left behind if the daemon was killed rather than stopped
- if the folder ever passes **500 MB**, the oldest go first

A turn is refused if one file is over **10 MB**, or its files together exceed
**25 MB** — you get a clear error rather than a silent truncation.

Two things that follow from this being turn-scoped: an agent may mention a file
it can no longer open (it should ask for a fresh copy — that is expected, not a
fault), and an agent whose `allowed_tools` omits `Read` cannot open these at all.
`humanikd verify` warns about the second.

All of it is adjustable in `config.yaml` — see CONFIG.md.

## What's new in v0.1.10

**Windows works.** Every job on a Windows device previously failed the moment it
started, with `The filename or extension is too long`. That message names the
path, but the cause was the size of the request — it went on the command line,
and Windows caps that at 32,767 characters. It now goes over stdin, which has no
such limit.

Four more Windows fixes came with it:

- **Claude Code is found when it's installed via npm.** The lookup wanted a
  literal `claude.exe`; npm ships `claude.cmd`. It now honours PATHEXT and also
  checks npm's global folder.
- **Auto-start actually stays up.** Older builds registered a system service that
  could not read your account's credentials, so it started and stopped within
  seconds, every time. `humanikd service install` now creates a **logon task**
  that runs as you — no admin, no password.
- **Readable output.** `←[1m` and `Γ£ô` no longer appear in PowerShell.
- **A real install command.** The `curl … | sh` line cannot run in PowerShell;
  the console now offers per-platform commands, and this README documents the
  Windows one.

> **Upgrading a Windows machine from v0.1.9 or earlier?** Remove the old, broken
> service once — from an elevated PowerShell: `sc.exe delete humanikd` — then run
> `humanikd service install` normally.

**Files a job brings with it.** Turns can now carry a screenshot or a document.
The daemon writes them into the run's own folder, tells the agent where they are,
and cleans them up on its own — see [What this writes to your
disk](#what-this-writes-to-your-disk).

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
