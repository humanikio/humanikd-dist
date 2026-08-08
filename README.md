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

**Windows** — use PowerShell. (The line above cannot work here: PowerShell
aliases `curl` to `Invoke-WebRequest`, which rejects `-fsSL`.)

```powershell
irm https://github.com/humanikio/humanikd-dist/releases/latest/download/install.ps1 | iex
humanikd setup   # guided first-run — START HERE
```

Installs to `%LOCALAPPDATA%\Programs\humanikd` and adds it to your PATH — **no
admin needed**. The installer updates the window you ran it in, so `humanikd
setup` works immediately; other open windows won't see it until you reopen them,
because Windows reads PATH when a shell starts.

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
| `serve.log` | What the daemon is doing — **v0.1.13+**, written however it was started |
| `ws/<tenant>/<workspace>/` | Working directory for agent runs, one per workspace |

`serve.log` is the first place to look when a background daemon is misbehaving.
Before v0.1.13 one started by the Windows logon task wrote nowhere at all — a
task action has no shell to redirect from — so "it is running but nothing
happens" had no evidence to inspect.

### Files a job brings with it

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

## What's new in v0.1.14

### A device can now drive your browser — if you turn it on

A Claude Code device can open pages, click, type, read the console and fill forms
in **your** real browser, using the sessions you are already signed into. No OAuth
to set up, no connector to configure.

**It is off by default and stays off until you say otherwise.** `humanikd setup`
asks once, defaults to no, and writes the config for you if you say yes.

This is a different kind of permission from everything else on a device, and worth
one paragraph before you enable it. Every other tool acts on the machine, inside a
working directory the daemon derives — that directory is the blast radius, which is
why a read-only tool ceiling is a real protection. The browser is not bounded by
it. It acts as you, everywhere you happen to be logged in: mail, banking, cloud
consoles, your own admin panels. Nothing in `config.yaml` makes that smaller.

Turning it on takes **two** lines, and both are required:

```yaml
backend:
  claude_agent:
    allowed_tools: [Read, Grep, Glob, mcp__claude-in-chrome]
    chrome:
      enabled: true
```

`chrome.enabled` loads the browser tools; `allowed_tools` permits them. One
without the other is the quiet failure — the tools appear and every call is
refused. `humanikd verify` says which half is missing. Note the **hyphens**: unlike
connector names, this one is not normalised, and the underscore spelling matches
nothing while reading as configured.

**Four things you have to do by hand**, in this order: install the Claude
extension and sign in, run `claude --chrome` once, **quit and reopen your browser**
(it reads the handshake only at startup, and skipping this is the most common
failure by a wide margin), then grant the sites you want reachable in the
extension's own settings. `humanikd verify` re-checks the first three and lists
whatever is outstanding. It cannot see the fourth.

**If `ANTHROPIC_API_KEY` is set where the daemon can see it, the browser silently
does nothing.** Claude Code disables the integration for API-key credentials, with
no error. `verify` now looks in three places — the service definition, the launchd
session, and your shell — and names the one it found, because the remedy differs
and "the daemon's environment" is usually not the terminal you are standing in.

Concurrent jobs on one device **share one browser and are not isolated from each
other**: listing tabs returns every tab. There is no per-job profile. If jobs must
not see each other's browsing, dedicate a machine.

Devices set to `mcp.scope: isolated` refuse to enable the browser at all. Isolated
hides the connectors you configured, but it cannot hide the browser — that server
is built into Claude Code rather than configured in a file — so allowing both would
hide your Gmail while handing over your live session.

Full details: [Claude in Chrome](https://code.claude.com/docs/en/chrome) for the
extension itself, and CONFIG.md for the device side.

## What's new in v0.1.13

### ⚠️ Windows laptops — one command needed after upgrading

If you installed auto-start on **v0.1.10, v0.1.11 or v0.1.12**, run this once
after upgrading:

```powershell
humanikd service install
```

Those builds registered the logon task with Windows' own default settings, which
are written for an app you launch occasionally rather than a daemon. On a laptop
that meant it **would not start if you logged in on battery, was killed the moment
you unplugged the charger, and was killed again after 72 hours** — with nothing to
restart it. It reported healthy the whole time.

v0.1.13 turns all of that off explicitly. But **upgrading alone does not fix an
already-registered task** — the settings live in the task, not the binary. The
command above replaces it in place. Desktops on mains power were mostly unaffected.

### Everything else

**The agent can open the files it is given.** Turns that carry a screenshot or a
document now work end to end. The files were being written correctly all along;
the agent was handed a path it could not open and reported that nothing had
arrived. See [What this writes to your disk](#what-this-writes-to-your-disk).

**There is a log now.** `~/.humanikd/serve.log`, written however the daemon was
started. A background daemon on Windows previously wrote nowhere at all.

**A typo in `config.yaml` fails loudly.** Unknown or misplaced keys used to be
discarded in silence, so the machine ran on defaults while the file on disk said
otherwise. The reported case was `allowed_tools` written at the top level instead
of under `backend.claude_agent`: it parsed, nothing complained, and every tool
call was refused — indistinguishable from "MCP is broken". The daemon now names
the offending key and line at startup, and `humanikd verify` fails rather than
reporting on a config that is not in effect.

**`humanikd upgrade` no longer tells a local build it is current.** If you built
from source, its version number outranked every release, so `upgrade` reported
nothing to do while the machine sat several releases behind.

> **`service install` says `Access is denied`?** That is a Task Scheduler policy
> on the folder, not something humanikd needs. v0.1.13 tries a subfolder first,
> which avoids it on most machines; if it still appears, run the command **once**
> from an Administrator PowerShell. The task it creates is still unprivileged.

> **Upgrading from v0.1.9 or earlier on Windows?** `humanikd service install` also
> removes the old system service those builds registered — the one that could never
> start. If it says it needs elevation, run `sc.exe delete humanikd` once as
> administrator.

## If something is wrong

Start with `~/.humanikd/serve.log` and `humanikd verify`. The rest of these are
the ones whose message points somewhere other than the cause.

| What you see | What it means |
|---|---|
| **Windows:** `The filename or extension is too long` | Not the path — Windows' 32,767-character limit on a whole command line. Fixed in v0.1.10; upgrade. |
| **Windows:** `The command line is too long` | A **different, lower** limit: `cmd.exe` caps at 8,191, and a `claude.cmd` shim runs under it. v0.1.13 prefers a real `claude.exe` when both exist. Installing Claude Code natively (`irm https://claude.ai/install.ps1 \| iex`) avoids it entirely. |
| The daemon runs but nothing happens, on a laptop | The logon task inherited Windows' battery and 72-hour limits. Re-run `humanikd service install` — see the v0.1.13 notes above. |
| `Claude Code is not installed` but `claude --version` works | Older builds looked only for `claude.exe`. Fixed in v0.1.13; or set `backend.claude_agent.binary` to the full path. |
| Tools are visible to the agent but every call is refused | The tool is not in `allowed_tools`. It belongs under `backend.claude_agent`, **not** at the top level of `config.yaml` — v0.1.13 refuses to start on the misplaced version rather than ignoring it. `humanikd verify` prints the exact names. |
| The agent mentions a file it cannot open | Expected. Files a turn brings are kept for that turn only; it should ask for a fresh copy. |
| `humanikd upgrade` says you are current, but you are not | You are on a build made from source. v0.1.13 says so instead. |

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
