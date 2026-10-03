# humanikd

The downloadable **device daemon** for HumanikOS. It runs on a machine you own,
dials out to the HumanikOS gateway, and serves work locally — a local model
(Ollama) or a local Claude Code agent. Billed as `byok` ($0 platform credits):
you bring the compute.

The **private key never leaves your machine.** The daemon generates an Ed25519
keypair at enrollment and sends only the public half.

An agent running here can also use the tools of the office it works for (its
contacts, calendars, files and integrations). Those tools run in the office, with
the office's credentials; this machine only sees each result. See
[Office tools on your machine](docs/office-tools.md).

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

**Your connectors follow the Claude account, not the machine.** `claude.ai`
connectors (Gmail, Drive, Calendar) disappear if you sign into a different
account. Run `humanikd verify` after any account change. Connecting a server is
not the same as allowing it: `allowed_tools` decides what the agent may call. See
[Security](docs/security.md).

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

## Learn more

| Page | What it covers |
|---|---|
| [How humanikd works](docs/how-it-works.md) | The outbound connection, how work arrives, sleep and reconnecting |
| [Office tools on your machine](docs/office-tools.md) | How an agent here uses the office's tools without holding their credentials |
| [Security](docs/security.md) | The tool limit, your key, connectors, the browser |
| [Files and logs](docs/files-and-logs.md) | Everything under `~/.humanikd`, and files a turn brings with it |
| [Troubleshooting](docs/troubleshooting.md) | Messages that point somewhere other than the cause |

Everything humanikd writes lives under `~/.humanikd/`. **Back up
`identity.json`**: it is this machine's enrollment key and cannot be reissued.
`~/.humanikd/serve.log` is the first place to look when something is wrong.

## What's new in v0.1.18

**`humanikd version` now tells you whether you are current.**

```
$ humanikd version
humanikd 0.1.17
update available: 0.1.17 → 0.1.18
upgrade in place with:
  curl -fsSL https://github.com/humanikio/humanikd-dist/releases/latest/download/install.sh | sh
```

Before, it printed the number and nothing else, so the question people actually
have needed a second command they had no reason to know about. A machine showing
an old version next to a console showing a new one looks correct, and an install
that quietly did not replace the binary was invisible.

The check is quick and optional. If this machine is offline or the check is slow,
the version still prints at once and nothing else is shown. `humanikd upgrade` is
still the command that reports a failed check, because there the check is the
whole job.

## What's new in v0.1.17

**An agent turn now gets two hours, not thirty minutes.** The default
`request_timeout` for both agent backends is now `2h`.

This machine is the outer bound for any job it runs: whatever HumanikOS allows, a
lower ceiling here is the one that ends the turn. At thirty minutes, long jobs
were being cut while they were still working. A measured browser run, sending
intros and writing 70 rows to a table, was stopped at exactly thirty minutes with
a third of its work unwritten.

Nothing else changes. A turn that is genuinely stuck is still caught much earlier,
because HumanikOS stops a job that goes quiet.

**If your `config.yaml` sets `request_timeout` yourself, it is unchanged.** Edit it
to `2h` and restart the daemon if you want the new ceiling:

```
humanikd service restart
```

## What's new in v0.1.16

**Cancelled jobs now stop.** When HumanikOS cancels a job, for example because the
office that asked for it gave up, the daemon now stops the work. Before, the cancel
was ignored and an agent turn kept running, editing files and calling tools, until
its own deadline of up to 30 minutes.

- A Claude Code run is stopped with SIGTERM, so it can end its shell commands
  cleanly. If it is still running 10 seconds later, it is killed. On Windows it is
  killed straight away, because Windows has no SIGTERM.
- An OpenClaw turn is stopped with `chat.abort`. Before, a turn the daemon stopped
  waiting for kept its place in the harness's queue, and later prompts on that
  conversation waited behind it until the harness restarted.
- A stopped job reports `cancelled`, and a job that ran out of time reports
  `timeout`. The two used to be reported the same way.

**Settings files in the agent's workspace are no longer loaded.** Claude Code now
runs with `--setting-sources user`. It reads your own settings in `~/.claude`, but
not `.claude/settings.json` or `.claude/settings.local.json` inside the workspace
directory. That directory belongs to the agent. On a device whose tool ceiling
allows `Write`, an agent could otherwise add a hook there, and the hook would run
shell commands on a later turn, outside the ceiling.

If you relied on project settings inside `~/.humanikd/ws`, move them into your user
settings.

## What's new in v0.1.15

Corrections to what v0.1.14 said about the browser, from running it. No behaviour
changed; what the daemon tells you did.

**The browser surface reaches past this machine.** v0.1.14's notes described "your
real browser" as though it meant the one on the device. Pairing is per Claude Code
account, so a browser on another computer is reachable from a job here. Measured on
a macOS device that listed a connected Windows browser.

`humanikd verify` now says **"on this machine"** where it means it. A missing local
browser reads as *not observed here* rather than *nothing is reachable*, because
`pgrep` cannot see another host and nothing outside the agent's own session can
enumerate paired browsers.

**More than one paired browser needs a human.** When two or more are paired, Claude
Code asks which to use before acting. A headless device has nobody to answer, which
makes this a second step that cannot be automated, alongside per-site permissions.
Pair one browser per account on any device meant to run unattended. What a headless
job does on hitting this is not yet measured, so `verify` states it as a caution
rather than a documented failure.

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

**It is not bounded by the machine either.** Browsers pair to your Claude Code
**account**, not to a host, so a browser signed in on a different computer — a
different operating system, even — is reachable from a job running on this device.
Enabling this scopes the capability to every browser paired to that account, not to
the box humanikd is installed on.

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

Start with `~/.humanikd/serve.log` and `humanikd verify`, then see
[Troubleshooting](docs/troubleshooting.md).

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
