# Models, frameworks and agents

humanikd lets Humanik send work to this computer. What does the work is one of
three kinds of thing, and the Humanik OS app has a page for each kind.

| Kind | What it is | On this computer today | App page |
| --- | --- | --- | --- |
| **Model** | A program that answers a prompt with text. It has no tools: it cannot open files, run commands or use a browser. | Ollama | Models |
| **Framework** | The agent program. It runs a whole job: it reads and changes files, runs commands, uses connectors and the browser, within the limits you set. | Claude Code, OpenClaw | Frameworks |
| **Agent** | An identity set up in Humanik: a name, instructions, and which framework and model it uses. | Not separate yet | Agents (coming later) |

They fit together like this: **an agent runs on a framework, and the framework
gets its answers from a model.** Claude Code, for example, gets its answers from
your Claude account.

## What this computer serves today

A computer serves one of the three programs at a time: Claude Code, Ollama or
OpenClaw. Choose it in the app under **This machine › Agent**, or in
`config.yaml`. Every job Humanik sends to this computer goes to that program.

Each program still has its own page, and you can set it up before you switch to
it.

## Pages in the app

| Page | What it covers |
| --- | --- |
| [Frameworks › Claude Code](claude-code.md) | Which Claude Code setup and account jobs use, connectors and MCP servers, the browser, time limits |
| [Frameworks › OpenClaw](openclaw.md) | Docker, the OpenClaw gateway, the model key, the folders the agent may see |
| [Models › Ollama](ollama.md) | Where Ollama is, which models it has, which ones Humanik may use |
| This machine › Access | What jobs may do on this computer: tools and the working folder. It applies to every framework |

Every section on the Models and Frameworks pages is tagged with who owns it:

- **Set by humanikd**: saved in humanikd's `config.yaml`. You change it on the
  page and save.
- **Set in Claude Code / Ollama / OpenClaw**: that program's own settings. The
  page shows them as they are now. Changes go through that program's own
  commands or website. humanikd never edits another program's files.

## Ways a model on this computer is used

| Route | Status |
| --- | --- |
| An office in Humanik uses this computer's Ollama as its model (BYOK, Local) | Works |
| An agent on this computer uses this computer's model, through Humanik | Works |
| Claude Code on this computer uses this computer's Ollama directly, without going through Humanik | Not yet. Shown as "Coming later" on the Claude Code page |

## Coming later

- **Several agents on one computer**, each with its own instructions, folders
  and tool list, and each accepted by you on this computer before it can run.
  They will appear on an Agents page.
- **More model programs** besides Ollama.
