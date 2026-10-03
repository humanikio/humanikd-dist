# Office tools on your machine

An agent running on your machine can use the tools of the office it works for: its
contacts, calendars, files and browsers, and the endpoints your integrations add. Those
tools run in the office, with the office's credentials. Your machine never holds a key for
them. This page explains how humanikd makes that work.

It applies to machines set up as a **Claude Code agent**.

## The bridge

Every office has an **office sandbox**: a machine HumanikOS runs for that office, holding
its tools, its integrations and their credentials. Its tools form one catalog.

When an employee's turn runs on your machine, the office's tool list comes with the turn:
each tool's name, description and parameters, and never a credential. humanikd then:

1. Starts a small **local MCP server** for that turn, named `hos`.
2. Starts Claude Code with that server attached. The agent sees each office tool as
   `mcp__hos__` followed by the tool's name, for example `mcp__hos__crm_list_contacts`.
3. When the agent calls one, humanikd sends the request up the same connection the turn
   arrived on. The office sandbox runs the tool and only the result comes back.
4. The agent reads the result and carries on in the same turn.
5. When the turn ends, the local server stops. Nothing about it is written to disk.

```
Claude Code ──MCP──▶ local server "hos" ──▶ humanikd ──▶ gateway ──▶ HumanikOS API ──▶ office sandbox
     ▲                                                                                     │
     └──────────────── result ◀── humanikd ◀── gateway ◀── HumanikOS API ◀──────────────────┘
```

The agent also keeps its own tools and the MCP servers you set up on this machine, so one
turn can read a local file and call an office tool.

## What gets bridged

| Kind | Example |
| --- | --- |
| Tools HumanikOS ships with every office | contacts and conversations, calendars, files, browsers, schedules |
| Your integration endpoints | Each endpoint an integration declares and enables becomes its own named tool |

Every office also has one general tool, `hos_integration_request`, which calls a connected
integration by its name and a path. It covers endpoints nobody declared, so the named tools
are better when they exist.

**Not bridged:** tools that belong to the office's own agent framework, such as its shell.
They only work inside the office.

## What your config controls, and what it does not

`allowed_tools` in `~/.humanikd/config.yaml` is the limit on what an agent may do **to this
machine**: read files, run commands, use the MCP servers you configured here.

**It does not apply to office tools**, and they do not need to be listed. They never touch
your machine: they run in the office, under the office's own permissions. If the job brings
office tools, humanikd bridges them, and there is nothing to configure.

## Limits today

- **The tool list arrives with every turn.** Nothing is kept between turns, so a new
  integration appears on the next turn after the office restarts. During one turn the list
  does not change.
- **One office tool call is limited to about 30 seconds.** A longer call fails with an
  error the agent can read, and the turn continues.
- **Pictures arrive when the tool returns one.** A picture attached to a chat message, for
  example, reaches the agent as an image along with the tool's text. Very large pictures are
  left out and the text says so. Needs humanikd v0.1.12 or later.
- **A dropped connection ends the turn**, including any tool call in progress.

## Related

- [How humanikd works](how-it-works.md)
- [Security](security.md)
- Product docs: [How a tool call travels](https://docs.humanik.io/docs/devices/tool-calls)
