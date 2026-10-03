# Files and logs

Everything humanikd writes lives under `~/.humanikd/`. Nothing is written outside it.

| Path | What it is |
| --- | --- |
| `identity.json` | This machine's enrollment key. **Back it up.** It cannot be reissued; losing it means re-enrolling as a new machine |
| `config.yaml` | Your settings. Safe to edit; restart the daemon afterwards |
| `serve.log` | What the daemon is doing (v0.1.13 and later), however it was started. Rotated at 8 MB to `serve.log.1` |
| `ws/<tenant>/<workspace>/` | The working folder for agent turns, one per workspace. Conversations in the same workspace share it |

## Config is read once

humanikd reads `config.yaml` when it starts. Editing the file changes nothing until you
restart it:

```bash
humanikd verify            # check the new settings first
humanikd service stop
humanikd service start
```

A misspelled or misplaced key stops the daemon at startup and names the key and line.

## Moving to a new computer

Copy `identity.json` as well as `config.yaml`. Copying only the config creates a second
machine. Retire the old record in the console afterwards.

## Files a turn brings with it

Some turns carry a file: a screenshot, a PDF, a spreadsheet. Claude Code takes text only, so
humanikd writes these files into the turn's working folder and tells the agent where they
are:

```
~/.humanikd/ws/<tenant>/<workspace>/.attachments/<job>/frame.png
```

You never need to clean this up:

- anything untouched for **2 hours** is removed, checked every 15 minutes
- during a long conversation, only the **last 3 turns** of files are kept
- **every start empties the folder**, which also clears anything left after a crash
- above **500 MB**, the oldest go first

A turn is refused if one file is over **10 MB**, or its files together exceed **25 MB**.

An agent may mention a file it can no longer open. That is expected: it should ask for a
fresh copy. An agent whose `allowed_tools` does not include `Read` cannot open these files
at all; `humanikd verify` warns about it.

## Related

- [Troubleshooting](troubleshooting.md)
- [Security](security.md)
