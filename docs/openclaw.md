# OpenClaw

OpenClaw is a **framework**: an agent that runs whole jobs. On this computer it
runs inside a Docker sandbox and sees only the folders you share with it. See
[Models, frameworks and agents](models-and-frameworks.md) for how it fits with
the rest.

In the app: **Frameworks › OpenClaw**.

## What humanikd does with it

humanikd runs OpenClaw's gateway as a service on this computer, with its own
settings folder, `~/.humanikd/openclaw`. Jobs from Humanik go to that gateway,
and the agent runs in Docker.

Use the `humanikd gateway` commands rather than plain `openclaw gateway`
commands: the plain ones would set up a gateway that reads a different settings
folder.

## What it needs

| Step | Command | Why |
| --- | --- | --- |
| Docker running | Docker Desktop | The agent runs in a container. humanikd refuses OpenClaw jobs without it |
| Gateway service | `humanikd gateway install` | Runs OpenClaw's gateway as a service, so it survives a restart |
| Sandbox image | `humanikd gateway sandbox` | Builds the container image once on this computer |
| Model key | `humanikd gateway auth` | OpenClaw needs a key for its model. An API key in your shell does not reach the service; it has to be added with this command |
| Check | `humanikd gateway status`, `humanikd gateway models` | Whether the gateway runs, and which model providers are missing a key |

The page shows whether Docker is running, and lists these commands with a copy
button.

## Folders the agent can see

The sandbox sees nothing of this computer until you share a folder. Each one is
written as `this-computer-path:sandbox-path`, with `:ro` at the end for read
only:

```yaml
backend:
  openclaw:
    binds:
      - ~/projects/acme:/workspace/acme
      - ~/Documents/notes:/workspace/notes:ro
```

Add and remove them on the page. Some folders can never be shared, and humanikd
refuses to start with one of them: your whole home folder, `~/.humanikd` (this
computer's identity key), credential folders such as `~/.ssh`, `~/.aws` and
`~/Library/Keychains`, Docker's own files, and system folders. A shared folder
may not be inside one of them or contain one.

## What it cannot do

The sandbox has no access to apps on this computer or to your local network, so
OpenClaw skills that need macOS apps (Notes, Reminders, iMessage), screen
recording, or devices on your network do not work here. Sharing more folders
does not change that.

## When Humanik uses it

Humanik sends OpenClaw jobs here only while this computer serves OpenClaw (This
machine › Agent). The default time limit for a job is 2 hours.
