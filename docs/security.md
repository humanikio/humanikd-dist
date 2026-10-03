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
- `["*"]` removes the limit, for a machine whose only job is serving agents. Jobs then have
  to name the tools they need. It cannot be mixed with named tools.
- `bypassPermissions` is always refused, whether it comes from your config or a job.

### Where an agent can reach

| Surface | Runs where | Covered by `allowed_tools` |
| --- | --- | --- |
| Built-in tools (`Read`, `Bash`, `Write`) | This machine, in a working folder humanikd chooses per workspace | Yes |
| MCP servers you set up on this machine (Gmail, Linear…) | Wherever that server points | Yes |
| Office tools (`mcp__hos__*`) | The office sandbox, with the office's credentials | No, they never touch this machine. See [Office tools on your machine](office-tools.md) |
| Your browser (opt in) | Your real browser, as you | Yes, plus its own setting |

## MCP servers you set up here

With `mcp.scope: inherit` (the default), an agent sees the MCP servers you connected on this
machine. Seeing is not using: a server's tools can only be called if `allowed_tools` names
them. Name the server rather than each tool, for example `mcp__claude_ai_Gmail`.
`humanikd verify` prints the exact names for your servers.

Use `mcp.scope: isolated` on a machine that serves people other than you, so they do not
get your connectors.

**Connectors follow the Claude account, not the machine.** `claude.ai` connectors (Gmail,
Drive, Calendar) belong to the Claude account this machine is signed into. Sign into a
different account and they disappear, with nothing warning you. Servers you add yourself
with `claude mcp add` stay. Run `humanikd verify` after any account change.

## The browser (opt in)

A Claude Code agent can drive your real browser, using the sessions you are signed into.
It is off by default, needs two settings, and acts as you everywhere you are logged in, on
any browser paired to the same Claude account. It cannot be combined with
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
