# Troubleshooting

Start with two things:

```bash
humanikd verify        # checks everything this machine's role needs, spends nothing
tail -50 ~/.humanikd/serve.log
```

The rows below are the problems whose message points somewhere other than the cause.

## The machine

| What you see | What it means |
| --- | --- |
| The machine shows offline in HumanikOS | humanikd is not running or not connected. Check `humanikd service status` and the log |
| `enrolled: YES, but the key is UNREADABLE` | The daemon runs as a different account from the one that enrolled. Run it as that account. Do not re-enroll |
| A config edit changed nothing | Config is read once at startup. Run `humanikd service restart` |
| `field <name> not found in type config.fileConfig`, then `RUNNING ON DEFAULTS` | This humanikd is older than your settings: a newer version (or the app) wrote a setting it does not know. Install the latest release. From v0.1.24 the installer and the app refuse to install a version that cannot read your settings |
| `Load failed: 5: Input/output error` when starting the service (macOS) | humanikd kept failing to start, and macOS still holds the service while it waits to retry. Run `launchctl kickstart -k gui/$(id -u)/humanikd`. From v0.1.24, `humanikd service start` does this itself |
| The daemon stops at startup naming a key | A misspelled or misplaced setting in `config.yaml`. Most often `allowed_tools` written at the top level instead of under `backend.claude_agent` |
| `humanikd upgrade` says you are current, but you are not | You are on a build made from source |
| A notification says humanikd is out of date | A newer release exists. Run the command in the notification, or `humanikd upgrade` to see it. The daemon checks each time it starts and shows this at most once every 12 hours for the same release |
| No update notification, but `serve.log` says out of date | The daemon runs as a system service, which has no desktop to show it on, or this is a build made from source, which only logs it |

## Claude Code agents

| What you see | What it means |
| --- | --- |
| Jobs fail with `authentication_failed` or `auth_required` | This machine is not signed in to Claude Code. Run `claude` once and sign in |
| Tools are visible but every call is refused | The tool is not in `allowed_tools`. `humanikd verify` prints the exact names for your MCP servers |
| A connector that worked yesterday is gone | The machine is signed into a different Claude account. `claude.ai` connectors belong to the account |
| The agent mentions a file it cannot open | Expected. Files a turn brings are kept for that turn; it should ask for a fresh copy |
| An office tool fails after about 30 seconds | The current limit for one office tool call. The agent sees the error and continues |
| An office tool says `unknown tool` | The integration endpoint is not declared and enabled, or the office has not restarted since it was added |
| An office tool returned no picture | The picture was over the size limit (the tool's text says so), or humanikd is older than v0.1.12 |
| `refusing to run: …/.mcp.json exists` | The agent wrote an MCP config into its working folder. Remove it, or move those servers into your own Claude Code config |

## Windows

| What you see | What it means |
| --- | --- |
| `The filename or extension is too long` | Windows' limit on a whole command line. Fixed in v0.1.10; upgrade |
| `The command line is too long` | A lower limit (8,191 characters) that applies when Claude Code is installed as `claude.cmd`. Install it natively with `irm https://claude.ai/install.ps1 \| iex` |
| The service installs, starts, then stops | An old build registered a system service that cannot read your key. Run `humanikd service install` on v0.1.10 or later |
| `service install` says `Access is denied` | A Task Scheduler folder policy. Run the command once from an Administrator PowerShell; the task it creates is still unprivileged |
| Healthy but never runs, on a laptop | A task installed by v0.1.10 to v0.1.12 kept Windows' battery and 72-hour limits. Upgrade and run `humanikd service install` again |

## The Humanik OS app

| What you see | What it means |
| --- | --- |
| "Something went wrong in Humanik OS" with an error and **Copy details** | A screen in the app failed. Try **Try again**, then **Reload the app**. If it comes back, use **Copy details** and send us the text. humanikd is not affected and keeps serving |
| "Application error: a client-side exception has occurred" | The same kind of failure in app v0.1.21, which did not show the error. Run the install command again to get the current version, which does |
| On Windows, a terminal window opens for a moment every few seconds | App v0.1.21 opened one each time it checked humanikd's status. Fixed in v0.1.22: run the install command again |
| The app offers to "update" humanikd to a LOWER version | You are on a test build (a version with a dash, such as 0.1.23-dev.1). Do not update; wait for the final release. Fixed in v0.1.24, which never offers a test build an older release |
| No notifications | See [Notifications](notifications.md#if-notifications-do-not-appear) |

## Local models

| What you see | What it means |
| --- | --- |
| The reply is empty | The model ignored the tool definitions. Use a tool-capable model and run `humanikd verify` |
| `model_not_found` | The model is not pulled on this machine, or not in your `models` list |
| `backend_unavailable` | Ollama (or your compatible server) is not running at the configured URL |

## Related

- [The Humanik OS app](desktop-app.md)
- [Files and logs](files-and-logs.md)
- [How humanikd works](how-it-works.md)
