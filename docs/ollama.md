# Ollama

Ollama is a **model**: it answers a prompt with text. It has no tools, so it
cannot open files, run commands or use a browser. See
[Models, frameworks and agents](models-and-frameworks.md) for how it fits with
the rest.

In the app: **Models › Ollama**.

## What humanikd does with it

**humanikd never starts, stops or installs Ollama.** It only talks to the address
you give it:

- to list the models Ollama has, so Humanik can offer them, and
- to send each request from Humanik and stream the answer back.

Ollama itself loads a model into memory on its first request and frees it after
a few idle minutes. If nothing answers at the address, requests to this computer
fail until Ollama runs.

The page has a **Start Ollama** button when it is not running. It opens the
Ollama app on this computer (macOS and Windows). On Linux, start it with
`ollama serve` or your service manager.

## Where Ollama is

| Setting | What it does | Default |
| --- | --- | --- |
| Address | Where Ollama answers. Can be another computer on your network | `http://localhost:11434` |
| Longest an answer may take | Includes loading the model into memory the first time | 5 minutes |

**Test** asks Ollama for its version and shows how long it took. It checks the
saved address.

In `config.yaml`:

```yaml
backend:
  ollama:
    url: http://localhost:11434
    request_timeout: 5m
    models: []        # empty = every model Ollama has
```

## Models

The page lists every model Ollama has, with its size and whether it is in memory
right now. Each one has an **Offered to Humanik** switch.

- With every switch on, `models` stays empty in `config.yaml`, which means every
  model Ollama has, including ones you add later.
- Switching some off writes the ones that stay on, and **exactly that list is
  offered**, even a model Ollama does not have (a request for it then fails with
  "model not found"). The page shows such a model as "Listed, not installed" so
  you can switch it off.

Add models with `ollama pull <name>` in Terminal.

## When Humanik uses it

Humanik sends model requests here only while this computer serves Ollama (This
machine › Agent). Choosing Ollama there runs humanikd as a **system service**
instead of under your account, so your computer asks for an administrator
password when you save. A system service has no desktop, so it cannot show
notifications.

**Used by**:

| | |
| --- | --- |
| Offices in Humanik | Offices whose model is set to this computer (BYOK, Local) send their requests here through Humanik. The page will list them in a later version |
| Frameworks on this computer | Claude Code using this model directly is coming later |
