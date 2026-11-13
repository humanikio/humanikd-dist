# Security

What humanikd can and cannot do on your computer, what HumanikOS can and cannot ask of it,
and the settings that change that.

## The one rule

**Your machine declares a limit. A job may ask for less, and never more.**

The limit lives in `~/.humanikd/config.yaml`, written by you. A job can ask for a smaller
set of tools; humanikd grants only the tools that are both asked for and allowed here. The
folder an agent works in is chosen by humanikd, not by the job. If HumanikOS were
compromised, it still could not widen what an enrolled machine allows.

## The identity key

- Created on your machine at enrollment. Only the public half leaves it.
- Stored in `~/.humanikd/identity.json`, encrypted by your operating system's keystore
  where one is available. `humanikd key status` shows how it is held.
- The encryption belongs to the account that enrolled. Run the daemon as that same
  account.
- **Back the file up.** Losing it means re-enrolling, which creates a new machine record.

## The tool limit (Claude Code agents)

```yaml
backend:
  claude_agent:
    enabled: true
    permission_mode: dontAsk
    allowed_tools: [Read, Grep, Glob]   # the limit; this is the default
```

- The default is read-only.
- `Write`, `Edit` and `Bash` are available but off until you add them. Prefer narrow
  forms such as `Bash(git status *)` over `Bash`.
- `["*"]` (Everything in the app) removes the limit, for a machine whose only job is
  serving agents. Jobs get every built-in tool, every connector and MCP server of the
  Claude Code setup, and the browser when it is on. A job that names the tools it wants
  gets only those. It cannot be mixed with named tools.
- **Folders jobs may use.** When an office uploads a file from your machine to a bucket,
  or downloads one to it, humanikd moves the file itself, only inside these folders (by
  default the job's own folder). Two switches, off by default, also hold every file
  tool (a Claude Code hook) and every command (Claude Code's sandbox, not on Windows)
  to the same folders. `~/.humanikd` outside its job folders, `~/.ssh` and Claude
  Code's sign-in files are always refused. See CONFIG.md, `local_files`.
- `bypassPermissions` is always refused, whether it comes from your config or a job.

### Where an agent can reach

| Surface | Runs where | Covered by `allowed_tools` |
| --- | --- | --- |
| Built-in tools (`Read`, `Bash`, `Write`) | This machine, in a working folder humanikd chooses per workspace | Yes |
| MCP servers you set up on this machine (Gmail, Linear…) | Wherever that server points | With a separate setup for jobs, yes. With your own Claude Code setup, your Claude Code settings decide instead (see below) |
| Office tools (`mcp__hos__*`) | The office sandbox, with the office's credentials | No, they never touch this machine. See [Office tools on your machine](office-tools.md) |
| Your browser (opt in) | Your real browser, as you | Yes, plus its own setting |

## MCP servers you set up here

With `mcp.scope: inherit` (the default, shown in the app as **Humanik's and mine**), an
agent sees the MCP servers you connected on this machine, next to the tools Humanik sends.
Seeing is not using: a server's tools can only be called if `allowed_tools` names them.
Name the server rather than each tool, for example `mcp__claude_ai_Gmail`.
`humanikd verify` prints the exact names for your servers.

**With your own Claude Code setup, Claude Code's settings decide MCP tools.** Claude Code
adds the `permissions.allow` list in the setup's `settings.json` to every job. Claude Code
documents this: in `dontAsk` mode, tools pre-approved by `permissions.allow` rules still
run ([Claude Code: permission modes](https://code.claude.com/docs/en/permissions#permission-modes)).
So with `claude_dir: mine` (the default), humanikd does not pass the `mcp__` rules in
`allowed_tools` (except the browser's): which servers jobs may use is whatever
`~/.claude/settings.json` allows. In the app, the **Jobs may use** switches then change that
file (only its `permissions.allow` list, with a backup), which also changes what Claude Code
in Terminal may use. The rules stay in `config.yaml` and apply again with
`claude_dir: own`. If you want humanikd to be the limit for MCP tools, choose **Separate
setup for jobs** (`claude_dir: own`); `humanikd verify` and the app warn if that setup's own
`settings.json` allows a server too. See
[Claude Code](claude-code.md#which-claude-code-setup-jobs-use).

Use `mcp.scope: isolated` (**Humanik's only**, offered with a separate setup) on a machine
that serves people other than you, so they do not get your connectors. It passes Claude Code's `--strict-mcp-config`,
which leaves out every connector, plugin server and MCP server of your own. Humanik's tools
are still sent.

**Connectors follow the Claude account, not the machine.** `claude.ai` connectors (Gmail,
Drive, Calendar) belong to the Claude account this machine is signed into. Sign into a
different account and they disappear, with nothing warning you. Servers you add yourself
with `claude mcp add` stay. Run `humanikd verify` after any account change.

## The browser (opt in)

A Claude Code agent can drive your real browser, using the sessions you are signed into.
It is off by default, needs two settings, and acts as you everywhere you are logged in.
Browsers belong to your Claude account, so humanikd links jobs to the Chrome on this
computer (see [Claude Code](claude-code.md#link-this-computers-chrome)). A check that
humanikd adds to every browser job refuses any browser action until the job has selected
that Chrome, and refuses other browsers outright (see
[How browser jobs are checked](claude-code.md#how-browser-jobs-are-checked)). It cannot be combined with
`mcp.scope: isolated`. Read the product docs page before turning it on:
[Let jobs use your browser](https://docs.humanik.io/docs/devices/browser).

## What humanikd never does

- Holds a credential for your Claude account, an office, or an integration. Claude Code
  signs in on its own, and office tools run in the office.
- Installs software for you. It prints the command.
- Lets a job choose its working folder, or grant a tool your config does not allow.
- Loads MCP servers an agent wrote into its own working folder.

## Related

- [How humanikd works](how-it-works.md)
- [Files and logs](files-and-logs.md)
- Product docs: [Connect your own machine](https://docs.humanik.io/docs/devices)
