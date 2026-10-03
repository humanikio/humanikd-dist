# How humanikd works

humanikd is a small program that runs on a computer you own. It lets HumanikOS send work
to that computer: a turn for an AI employee to run with Claude Code, or a completion for
a local model. This page explains the connection, how work arrives, and what happens when
your computer sleeps.

## The connection goes out, never in

HumanikOS never connects to your computer. humanikd opens one outbound connection to the
HumanikOS gateway and keeps it open while it runs. Work arrives down that connection.

There is no port to open on your router, and nothing on the internet can reach your
computer through humanikd.

## Proving who the machine is

When you enroll, humanikd creates a key pair on your computer. Only the public half is
sent to HumanikOS. The private half stays in `~/.humanikd/identity.json`, encrypted by
your operating system's keystore where one is available (macOS Keychain, Windows DPAPI,
or `systemd-creds` on Linux).

Every time humanikd connects:

1. The gateway sends a fresh random challenge.
2. humanikd signs it with the private key.
3. The gateway checks the signature against the public key, and checks that the machine
   has not been revoked.

The challenge is new every time, so a captured signature is useless later. If a machine is
revoked in the console, its connection is closed and it is refused when it tries again.

## How work arrives

```
HumanikOS API ──▶ Redis queue ──▶ gateway ──▶ humanikd ──▶ Claude Code or your local model
      ▲                                           │
      └──────── turn channel ◀── gateway ◀────────┘   output, tool requests, results
```

- **Down:** a job is placed on a Redis queue kept for the gateway instance holding your
  connection. That gateway reads it and sends it down the connection.
- **Up:** everything humanikd sends back, the agent's output and any tool requests, goes
  out on a channel kept for that one job, which the HumanikOS API is already listening on.

humanikd and the gateway pass this through without reading it. The HumanikOS API turns
the agent's output into the employee's reply.

## A machine serves one kind of work

Each machine is set up for one role, chosen in `humanikd setup`:

| Role | What it runs |
| --- | --- |
| Claude Code agent | A full agent turn. The agent can use tools on this machine, within the limit you set, and the office's tools through the bridge (see [Office tools on your machine](office-tools.md)) |
| Local model | Text completions from Ollama or a compatible server. No tools run on the machine |

## Sleep, shutdown and reconnecting

HumanikOS does not check whether your machine is on. The gateway refreshes a short-lived
record while your connection is open, and the record expires on its own about 90 seconds
after the connection stops.

| What happens | Result |
| --- | --- |
| You open the lid | humanikd reconnects and the machine is available within seconds |
| You close the lid, lose network, or the machine crashes | The record expires and the machine shows as offline |
| HumanikOS deploys the gateway | humanikd is asked to reconnect and does so, starting at about a second |

Work for an employee whose machine is offline falls back to the cloud. It never moves to
another of your machines.

**A turn in progress is not resumed.** If the connection drops in the middle of a long
agent turn, the rest of that turn is lost and the work has to be asked for again.

## Related

- [Office tools on your machine](office-tools.md)
- [Security](security.md)
- [Files and logs](files-and-logs.md)
- [Troubleshooting](troubleshooting.md)
- Product docs: [How work reaches your machine](https://docs.humanik.io/docs/devices/how-work-reaches-it)
