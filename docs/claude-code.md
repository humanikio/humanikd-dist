# Claude Code

Claude Code is a **framework**: it runs a whole job on this computer. humanikd
starts it once for each job, with the tools, folder and time limit you set, and
stops it when the job ends. See [Models, frameworks and agents](models-and-frameworks.md)
for how it fits with the rest.

In the app: **Frameworks › Claude Code**.

Claude Code is Anthropic's program, not part of humanikd. humanikd decides how each
job starts it (the command below). It changes one thing in Claude Code's own
files, and only when you press a switch: the list of allowed MCP servers in
`settings.json` (see [Connectors and MCP servers](#connectors-and-mcp-servers)). For
Claude Code itself (its settings, MCP, permissions, hooks), see
[Claude Code's docs](https://code.claude.com/docs/en/overview). This page links the right one in
each section, and so does the app.

## Which Claude Code setup jobs use

Claude Code keeps its sign-in, MCP servers and settings in a folder. You choose
which folder jobs use.

| Choice | Folder | What jobs get |
| --- | --- | --- |
| **My Claude Code** (default) | `~/.claude` | The Claude Code you use in Terminal: the same account and plan, connectors, MCP servers and settings. Also your hooks and your `CLAUDE.md` notes file |
| **Separate setup for jobs** | `~/.humanikd/claude` | Its own sign-in, MCP servers and settings. Your hooks and notes stay out of jobs, and the account can be a different one |

The choice also decides **who controls what**:

| | My Claude Code | Separate setup for jobs |
| --- | --- | --- |
| Built-in tools (Read, Bash, Write) | humanikd (This machine › Access) | humanikd (This machine › Access) |
| Browser on/off | humanikd, its own switch | humanikd, its own switch |
| Browser setup steps, linking this computer's Chrome | shared | shared |
| Which connectors and MCP servers jobs may use | **your Claude Code settings** (`permissions.allow` in `~/.claude/settings.json`). The **Jobs may use** switches change that file right away, so they apply to Claude Code in Terminal too | **humanikd**: a **Jobs may use** switch per server, saved with **Save** |
| Which MCP servers jobs start | always **Humanik's and mine** | you choose: **Humanik's and mine** or **Humanik's only** |
| Tools Humanik sends with a job | always in the job | always in the job |

**Each setup keeps its own choices.** Switching between **My Claude Code** and
**Separate setup for jobs** does not lose anything: the browser switch, and for
the separate setup its MCP servers and **Which MCP servers jobs start**, are
stored per setup and come back when you switch back. Built-in tools, the time
limit and the sign-in notice are shared by both. In `config.yaml` the per-setup
values are under `setups` (see [Configuration](#in-configyaml) below); the app
adds that block only when the two setups differ.

Why: with My Claude Code, Claude Code adds your own `permissions.allow` list to
every job, so a humanikd setting could only ever add to it, never limit it. So
there the switches change your Claude Code settings instead. Claude Code documents this under
[permission modes](https://code.claude.com/docs/en/permissions#permission-modes): in `dontAsk` mode,
tools allowed by `permissions.allow` still run.

### In config.yaml

```yaml
backend:
  claude_agent:
    claude_dir: own                    # mine (default) | own
    allowed_tools: [Read, Grep, Glob]  # shared by both setups
    setups:                            # only when the two setups differ
      mine:
        chrome: true
      own:
        chrome: false
        mcp_scope: isolated            # inherit | isolated
        mcp_allowed: [mcp__claude_ai_Gmail]
```

Only the setup `claude_dir` names is used; the other one's values are kept.
With `setups`, set the browser and MCP servers only there, not in
`chrome.enabled`, `mcp.scope` or `allowed_tools`; humanikd refuses a file that
sets them in both places.

Things to know about **My Claude Code**:

- Changing the account in the app also changes it for Claude Code in Terminal,
  because it is the same setup.
- **Hooks run during jobs**, and they are not limited by the tool settings under
  This machine › Access. The page tells you how many hooks jobs pick up.
- Your `~/.claude/CLAUDE.md` is added to every job.
- **Your Claude Code settings decide which connectors jobs may use.** Turning on
  **Jobs may use** for Gmail adds `mcp__claude_ai_Gmail` to `permissions.allow`
  in `~/.claude/settings.json`
  ([Claude Code: permissions](https://code.claude.com/docs/en/permissions)). What that means:
  - It takes effect at the next job. There is no **Save** for it.
  - **Claude Code in Terminal uses the same list**, so Gmail is then allowed there
    too, without asking. Turning it off removes it there too, including rules you
    wrote for single Gmail tools.
  - The app changes only that list, and only for MCP servers. Everything else in
    the file stays as it was. The file as it was before each change is kept as
    `settings.json.humanikd-backup`.
  - You can still edit the file yourself; the page reads it each time it opens.
- **Connector switches you set for the separate setup are kept, not used.** The
  page lists them under the connectors, with a button to allow them in your
  Claude Code settings. They apply again if you switch back.

After choosing **Separate setup for jobs** and saving:

1. Sign in under **Account** on the same page.
2. If you want your MCP servers there too, press **Copy my MCP servers**. It
   copies the list of servers, not their sign-ins: servers that need one show a
   **Sign in** button.

Version note: `claude_dir` is new in humanikd 0.1.25. Older versions refuse a
`config.yaml` that has it, so the app only writes it when you choose the
separate setup.

Claude Code's docs: [settings files](https://code.claude.com/docs/en/settings),
[hooks](https://code.claude.com/docs/en/hooks), [CLAUDE.md](https://code.claude.com/docs/en/memory).

## Account

The page shows the account jobs run as: email, organization, plan, and how it
is signed in. The buttons run Claude Code's own commands:

| Button | Command it runs |
| --- | --- |
| Sign in / Change account | `claude auth login`. Your browser opens; the page updates when you finish there |
| Sign out | `claude auth logout` |
| Sign in with the Console | `claude auth login --console`: pay per use with an API key instead of a plan |

The app does not handle your sign-in itself and never sees your Claude password
or tokens. Claude Code opens the browser, and Claude Code keeps the sign-in.
While the browser is open, the button changes to **Cancel**, which stops the
sign-in (use it if you closed the browser tab).

**Your browser decides the account.** Sign-in uses whichever Claude account your
default browser is signed in to on claude.ai. Check that before pressing
**Change account**.

**With a Console (API key) sign-in, connectors and browser jobs do not work.**
They need a sign-in with a Claude account.

Claude Code's docs: [signing in](https://code.claude.com/docs/en/authentication).

## Connectors and MCP servers

Claude Code can use tools from other programs through MCP servers. A connector
is also an MCP server: the difference is where it is added and where its sign-in
is kept. The page shows three groups:

| Group | Added where | Sign-in kept | On your other computers |
| --- | --- | --- | --- |
| **Connectors** (Gmail, Google Drive, Sheets, Calendar) | claude.ai, Customize › Connectors ([claude.ai/new#customize/connectors](https://claude.ai/new#customize/connectors), or **Manage connectors on claude.ai** in the app) | in your Claude account | yes, every computer signed in to the same account |
| **Your MCP servers** | `claude mcp add`, saved in `.claude.json` | on this computer (**Sign in** on the page runs `claude mcp login`) | no |
| **Plugin servers** | a plugin you installed in Claude Code | on this computer | only where the plugin is installed |

The list comes from `claude mcp list`. That command connects to each server, so
it can say "Connected" or "Needs sign-in". It takes a few seconds, so the list
loads after the rest of the page; press **Check again** to refresh it.

A server's config only says how to reach it. Which tools it has is asked from
the server each time Claude Code starts, so nobody types a list of tools.

**Seeing a server is not using it.** Each row says whether jobs may use it:

- **Separate setup for jobs:** each row has a **Jobs may use** switch. The switch
  adds the server's name to the tool list in `config.yaml` (for example
  `mcp__claude_ai_Gmail`). If the separate setup's own `settings.json` also
  allows a server, the page warns you, because that would allow it with the
  switch off, and offers to remove it from that file.
- **My Claude Code:** each switch shows and changes your Claude Code settings
  (see above), right away and for Terminal too.

If tools are set to Everything under This machine › Access, every server is
allowed.

**Tools Humanik sends with a job are always available to it.** An office's own
tools (its CRM, its email) run in Humanik with the office's sign-ins, and are
approved there. This page controls your computer's tools and accounts.

### Which MCP servers jobs start

This is a choice only with **Separate setup for jobs**. With My Claude Code, jobs
always start Humanik's tools and every server of yours. (If an older
`config.yaml` already says `isolated`, humanikd still applies it, and the page
shows it with a button to change it.)

| Choice | In `config.yaml` | What a job gets |
| --- | --- | --- |
| **Humanik's and mine** (default) | `mcp: { scope: inherit }` | Humanik's tools, and every connector, plugin server and MCP server above, like Claude Code in Terminal. Each one starts or connects for every job, and its tools are listed to the job, even when switched off. Switched-off ones still cannot be used |
| **Humanik's only** | `mcp: { scope: isolated }` | Humanik's tools only. None of the servers above. humanikd passes Claude Code's `--strict-mcp-config` |

This setting decides which servers are **in** a job. It does not decide which tools
may **run**: that is the switches and This machine › Access. Humanik's tools are in
every job either way.

Use **Humanik's only** on a computer that serves people other than you, so your
own accounts stay out of their jobs. It leaves out connectors too. It does not
leave out your hooks or your `CLAUDE.md`: choose **Separate setup for jobs** for
that. It cannot be used with the browser, so humanikd refuses that combination.

Three settings, three different things:

| Setting | Keeps out of jobs |
| --- | --- |
| Tools (This machine › Access), and the **Jobs may use** switches (separate setup) or your Claude Code settings (My Claude Code) | tools from running |
| **Which MCP servers jobs start** | servers from starting and being seen |
| **Which Claude Code setup jobs use** | your hooks, `CLAUDE.md`, your own MCP servers and plugins (connectors come with the account either way) |

Claude Code's docs: [MCP](https://code.claude.com/docs/en/mcp),
[how connectors reach Claude Code](https://code.claude.com/docs/en/mcp#how-connectors-reach-claude-code),
[plugins](https://code.claude.com/docs/en/plugins), [permissions](https://code.claude.com/docs/en/permissions).

## What humanikd starts for each job

For each job, humanikd starts Claude Code once, with this command. The page
shows it under **What humanikd starts for each job**, filled in from your
settings. **Show the whole command** lists every part with what it does.
**Copy command** copies a version you can run in Terminal yourself: it sends the
message "Say hello" the way a job sends its message, and leaves out the parts
that exist only during a job (the session, Humanik's tools, the browser note).

```text
claude -p
  --output-format stream-json --verbose
  --permission-mode dontAsk
  --setting-sources user
  --session-id <id>                    # or --resume <id> to continue a chat
  --tools Read,Grep,Glob               # built-in tools that exist in the job
  --mcp-config <Humanik's tools>       # only when the job brings office tools
  --strict-mcp-config                  # only with Humanik's only
  --chrome                             # only with the browser on
  --append-system-prompt <...>         # only with the browser on and linked
  --disallowedTools switch_browser     # only with the browser on
  --allowedTools Read Grep Glob mcp__claude_ai_Gmail
```

| Part | What it does |
| --- | --- |
| `-p` | Runs one job and exits. The job's message is sent on standard input, not on the command line |
| `--output-format stream-json --verbose` | Claude Code reports each step as it happens; humanikd passes the steps to Humanik unchanged |
| `--permission-mode dontAsk` | A tool that is not allowed is refused. Nobody is asked, because nobody is watching |
| `--setting-sources user` | Uses the setup's own settings, never settings files inside the job's folder, which a job could write |
| `--session-id` / `--resume` | One conversation per chat, continued by later messages |
| `--tools` | Which built-in tools exist in the job: your Tools list, without MCP names. `default` when Tools is Everything |
| `--mcp-config` | The office's tools, through a small server humanikd runs for the job |
| `--strict-mcp-config` | Leaves out every connector and MCP server of your own |
| `--chrome` | Starts Claude in Chrome |
| `--append-system-prompt` | Names this computer's linked Chrome. A job that picks another browser is stopped |
| `--disallowedTools` | `switch_browser` always; also `select_browser` until this computer's Chrome is linked |
| `--allowedTools` | The tools the job may use, narrowed to what the job asks for, plus Humanik's tools. Always last. With **Separate setup for jobs**: your Tools list and the connector switches. With **My Claude Code**: your Tools list and the browser only; connectors are not named here, because your Claude Code settings decide them |

Claude Code also adds the `permissions.allow` list from the setup's
`settings.json` to `--allowedTools`, in every job. The page shows that list under
the command.

With **Separate setup for jobs**, humanikd also sets `CLAUDE_CONFIG_DIR` to
`~/.humanikd/claude` for the job. The job runs in its own folder for its
workspace, and is stopped at **Longest a job may run**.

Claude Code's docs: [command-line flags](https://code.claude.com/docs/en/cli-reference),
[permission modes](https://code.claude.com/docs/en/permissions#permission-modes),
[running Claude Code from a program](https://code.claude.com/docs/en/headless).

## Browser (Claude in Chrome)

Jobs can use the real Chrome window on this computer, signed in as you, through
the Claude extension for Chrome. Turn it on with **Let jobs use Chrome**. The
page checks what Chrome needs: the extension installed and signed in, Chrome
able to reach Claude Code, Chrome open, and Claude Code signed in with a Claude
account.

### Link this computer's Chrome

Browsers, like connectors, belong to your Claude account. If the same account is
signed in to Chrome on two computers, a job could see both. So humanikd links
browser jobs to the Chrome on this computer:

- Press **Link this computer's Chrome** (or run `humanikd chrome link`). humanikd
  asks the Claude extension which browser is on this computer and saves it.
  Nobody is asked anything, on this computer or any other.
- The page shows the linked browser, for example "Browser 2 · macOS", and the
  other browsers on your account, which jobs here never use.
- From then on, every browser job is told to use only that browser. A job that
  tries to use another one, or to switch browsers, is stopped.
- Until a browser is linked, jobs cannot choose between browsers at all. With
  only one browser on the account nothing changes; with more than one, browser
  jobs stop instead of picking one.

Linking needs Claude Code 2.1.300 or newer. With an older Claude Code, Link
stops with a message saying so, and browser jobs are refused until Claude Code
is updated (see [Claude Code version](#claude-code-version)).

On Windows the page cannot always check "Chrome can reach Claude Code", because
Chrome keeps that setting in the registry there. The line then says it could not
check, instead of "not yet". If jobs can use Chrome, it is done.

Link again after reinstalling the extension or changing Chrome profile.
`humanikd chrome status` shows the link; `humanikd chrome unlink` removes it.

Claude Code's docs: [Claude in Chrome](https://code.claude.com/docs/en/chrome).

## Claude Code version

Each humanikd release names the oldest Claude Code it works with. For v0.1.26
that is **2.1.300**. Newer versions are fine: Claude Code updates itself, and
humanikd does not hold it back.

When this computer runs Claude Code jobs and its Claude Code is older, the app
says so in four places: this page, **This machine › Overview**, **This machine ›
Agent** and **This machine › Updates**. **Update Claude Code** runs Claude Code's
own `claude update`, the same as in a terminal, and shows its output. You can
also run `claude update` yourself.

Until it is updated, jobs that use the browser are refused with the reason, and
Link does not start. Jobs without the browser still run.

`min_version` in config.yaml can raise the minimum on one computer, never lower
it.

## Model

Claude Code picks the Claude model, on your plan (or your API key with a Console
sign-in). Using a model on this computer instead (Ollama) is shown as **Coming
later**.

## Program and limits

| Setting | What it does | Default |
| --- | --- | --- |
| Longest a job may run | A job still running after this is stopped | 2 hours |
| When Claude Code needs signing in again | Show a desktop notice, or do nothing | Show a notice |

What jobs may do on this computer (tools, the working folder, write access) is
on **This machine › Access**, because it applies to every framework.

## Checking from Terminal

```bash
humanikd status     # the "claude dir" and "browser" lines
humanikd verify     # Claude Code, sign-in, connectors and the browser, in detail
```
