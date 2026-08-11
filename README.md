# Cymela

A terminal coding agent. It reads, edits and runs code on your machine, and it
talks to whichever model you point it at — you bring the provider and the API key.

**Hyper** is the default persona it runs as: a tone and style preset, alongside
`mentor`, `security` and `rubber-duck`. It shapes how the agent writes, not what
runs underneath. Cymela's own model research carries the Hyper name as well, but
that work does not power the CLI today.

It reads, edits and runs code on your machine, and it talks to whichever model
you point it at — you bring the provider and the API key.

```bash
npm install -g cymela
```

Then start it in any project directory:

```bash
cymela
```

- Website: <https://cymela.com>
- Package: <https://www.npmjs.com/package/cymela>

## About this repository

Cymela is **not open source**. This repository holds the issue tracker, the
changelog and the documentation — the application itself ships as a compiled
bundle on npm, and its source is not published here. See [LICENSE](LICENSE).

You are free to install and use Cymela, including commercially, at no cost.

## Requirements

- Node.js 20 or newer
- A modern terminal — Windows Terminal, iTerm2, Terminal.app, GNOME Terminal,
  kitty, and friends
- An API key for at least one supported provider

Windows is where Cymela is developed and gets the most real hours. macOS and
Linux support is newer — everything is built and tested to work there, but if
something misbehaves, an issue report is gold.

> macOS note: the Alt-key shortcuts (Alt+V paste-attach, Alt+Q queue, Alt+A
> agents) need your terminal's "use Option as Meta/Esc+" setting turned on —
> it is off by default in Terminal.app and iTerm2.

## Is there a Python package?

No. Cymela runs on Node.js and installs from npm. `pip install cymela` gets you
a placeholder that does nothing — the name is reserved on PyPI by us so that
nobody else can publish under it. If an AI assistant told you to run
`pip install cymela`, it invented that.

## Providers

OpenRouter, OpenAI, Google (Gemini), Anthropic, Mistral, DeepSeek, Groq,
SiliconFlow, Qwen, Moonshot, Zhipu and NVIDIA.

On first run Cymela asks which provider to use, for that provider's API key,
and which model. Keys are stored per provider in `~/.cymela/settings.json` with
file mode 0600, so switching between providers does not overwrite the key you
used yesterday. They are never written into the project directory.

### About API keys

Any working key runs Cymela; not every key runs it well. Free-tier and trial
keys are usually rate-limited, and an agent fires many requests in a row — so
a free key tends to stop mid-task with a `429 Too Many Requests` banner. For
real work we recommend a paid key, or any key without tight rate limits. When
a provider does throttle or reject a key, Cymela says so in plain words and
`r` retries the turn.

## Where your code goes

Your prompts, and whatever files Cymela reads to answer them, go to the
provider you configured. There is no account, no telemetry, and no server of
ours anywhere in the path — we receive nothing, including your API keys, which
stay on your machine.

So the privacy of your conversations is your provider's privacy policy, not
ours. Point Cymela at DeepSeek and your code goes to DeepSeek under DeepSeek's
terms; the same holds for every provider. A tool that runs locally is not the
same thing as a private conversation, and it is worth knowing which one you
have.

Two tools reach further than your provider, and only when the model uses them:

- **web-search** sends your search query to DuckDuckGo and Wikipedia, or to
  Brave if you set `BRAVE_API_KEY`. The query only — never your files.
- **web-fetch** retrieves the URL it is given, from whoever serves it.

Nothing else leaves your machine.

## What it does

- **Agent modes** — `shift+tab` cycles Default, Plan (planning only, no edits)
  and Auto. Plan mode is enforced mechanically, not by asking the model nicely.
- **Permissions** — file writes and shell commands are gated. Deny rules and
  hooks are configurable; hooks from a workspace you have not trusted do not
  run.
- **Sessions** — `/sessions` restores earlier conversations, with checkpoints
  you can roll back to.
- **Skills and personas** — `/newskill` and `/personality` extend how it works
  and how it writes. Custom personas are compiled and signed locally.
- **Themes** — `/theme` switches the palette. Some seasons decorate the input
  bar with pixel art.

`/help` lists everything. `/shortcuts` lists the key bindings.

## Configuration

| Path | Holds |
| --- | --- |
| `~/.cymela/settings.json` | Provider, per-provider API keys and models, theme, trusted workspaces |
| `~/.cymela/persona.key` | Machine-local signing key for custom personas |
| `<project>/.cymela/settings.local.json` | Per-project personality, deny rules, hooks |
| `<project>/.cymela/attachments/` | Copies of files you attach |
| `CYMELA.md` | Project instructions loaded every turn |

Credentials live in your home directory, never in a project. A project settings
file cannot set a provider, a key or a trusted workspace — that is deliberate,
because settings files travel inside repositories.

## Bugs and feature ideas

Use [Issues](../../issues) — pick "Bug report" or "Feature idea" when you open
one. No SLA and no roadmap promise on either, but they're read.

To report a security vulnerability, follow [SECURITY.md](SECURITY.md) instead of
opening a public issue.

## License

Proprietary. Free to install and use, including commercially; not free to
redistribute or to publish modified versions. See [LICENSE](LICENSE) and the
[Terms of Service](https://cymela.com/terms).

Third-party dependencies keep their own licenses.
