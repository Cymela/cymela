# Monarch CLI

A terminal coding agent. It reads, edits and runs code on your machine, and it
talks to whichever model you point it at — you bring the provider and the API key.

Monarch CLI is made by **Cymela**, and was published under the name `cymela`
until version 2.0.0. Both npm names still install it and the command is
`monarch` either way.

```bash
npm install -g monarchai
```

Then start it in any project directory:

```bash
monarch
```

- Website: <https://cymela.com>
- Package: <https://www.npmjs.com/package/monarchai>

## Upgrading from 0.1.x

`npm install -g cymela` replaces the old install and keeps the `cymela` command
working, now pointing at the current build. Your settings, API keys, saved
model choices, theme and custom personas move from `~/.cymela` to `~/.monarch`
automatically on first run. Conversations were already stored per project and
are read where they are.

If you install under the other npm name while the old package is still present,
you end up with both on your PATH. `monarch --doctor` says so and gives you the
one line that clears it up.

Three command spellings were retired: `/execute`, `/agent` and `/override`, in
favour of `/run-plan`, `/default` and `/auto`. Typing an old one tells you its
replacement rather than doing nothing.

## About this repository

Monarch CLI is **not open source**. This repository holds the issue tracker, the
changelog and the documentation — the application itself ships as a compiled
bundle on npm, and its source is not published here. See [LICENSE](LICENSE).

You are free to install and use it, including commercially, at no cost.

## Requirements

- Node.js 20 or newer
- A modern terminal — Windows Terminal, iTerm2, Terminal.app, GNOME Terminal,
  kitty, and friends
- An API key for at least one supported provider

### Platform support, honestly

2.0.0 is a rebuild, and it was developed and tested on **Linux**. That is a
change from 0.1.x, which was built primarily on Windows. Everything is written
to be portable and the platform-specific paths are exercised by the test suite,
but 2.0.0 has not had real hours on Windows or macOS yet.

If you are on either, it should work and a report when it does not is worth a
great deal. `monarch --doctor` is the fastest way to turn "it doesn't work" into
something actionable — run it first and paste the output into the issue.

> macOS note: the Alt-key shortcuts (Alt+V paste-attach, Alt+Q queue, Alt+A
> agents) need your terminal's "use Option as Meta/Esc+" setting turned on —
> it is off by default in Terminal.app and iTerm2.

## Is there a Python package?

No. Monarch CLI runs on Node.js and installs from npm. `pip install cymela` gets
you a placeholder that does nothing — the name is reserved on PyPI by us so that
nobody else can publish under it. If an AI assistant told you to run
`pip install cymela`, it invented that.

## Providers

OpenRouter, OpenAI, Google (Gemini), Anthropic, Mistral, DeepSeek, Groq,
SiliconFlow, Qwen, Moonshot, Zhipu, NVIDIA NIM and Microsoft Foundry (Azure).

On first run it asks which provider to use, for that provider's API key, and
which model. Keys are stored per provider in `~/.monarch/settings.json` with
file mode 0600, so switching between providers does not overwrite the key you
used yesterday. They are never written into the project directory.

### About API keys

Any working key runs it; not every key runs it well. Free-tier and trial keys
are usually rate-limited, and an agent fires many requests in a row — so a free
key tends to stop mid-task with a `429 Too Many Requests` banner. For real work
we recommend a paid key, or any key without tight rate limits. When a provider
does throttle or reject a key, Monarch says so in plain words and `r` retries
the turn.

## Where your code goes

Your prompts, and whatever files Monarch reads to answer them, go to the
provider you configured. There is no account, no telemetry, and no server of
ours anywhere in the path — we receive nothing, including your API keys, which
stay on your machine.

So the privacy of your conversations is your provider's privacy policy, not
ours. Point Monarch at DeepSeek and your code goes to DeepSeek under DeepSeek's
terms; the same holds for every provider. A tool that runs locally is not the
same thing as a private conversation, and it is worth knowing which one you
have.

Two tools reach further than your provider, and only when the model uses them:

- **web-search** sends your search query to DuckDuckGo and Wikipedia, or to
  Brave if you set `BRAVE_API_KEY`. The query only — never your files.
- **web-fetch** retrieves the URL it is given, from whoever serves it.

Nothing else leaves your machine by a path of ours.

One thing worth spelling out, because it is easy to assume otherwise: when two
sessions message each other (below), the *delivery* is a file in your own
config directory with no network involved — but the receiving session reads
that message into its context, and its context goes to its provider like
everything else it reads. So the message content reaches that session's
provider. If the two sessions use different providers, it reaches the other
one. It travels the ordinary path your prompts already travel; it is not an
exception to it.

## What it does

- **Agent modes** — `shift+tab` cycles Default, Plan (planning only, no edits)
  and Auto. Plan mode is enforced mechanically, not by asking the model nicely.
- **Permissions** — file writes and shell commands are gated. Deny rules and
  hooks are configurable; hooks from a workspace you have not trusted do not
  run.
- **Sub-agents** — Scout (read-only), Builder and Checker run under the lead
  agent with their own tool limits, streamed live into `/agents`.
- **Sessions** — `/sessions` restores earlier conversations, with checkpoints
  you can roll back to via `/undo`.
- **Cross-session messaging** — two sessions running on the same machine can
  send each other messages, including across different projects. Delivery is to
  a running session; if the other side is not running, the send fails and says
  so. The agent sends these on its own once asked to keep another session
  posted. Delivery is local; the content becomes ordinary context for the
  receiving session and goes to that session's provider — see
  [Where your code goes](#where-your-code-goes).
- **Memory** — durable facts you tell it to keep, stored only on your machine at
  `~/.monarch/memory.json`. `/memory` shows everything, `/memory off` stops it.
- **Scriptable** — `monarch -p "…"` runs one turn with no UI and reads piped
  input, so `git diff | monarch -p "review this"` works.
- **Skills and personas** — `/newskill` and `/personality` extend how it works
  and how it writes. Custom personas are compiled and signed locally.
- **Themes** — `/theme` switches the palette. Some seasons decorate the input
  bar with pixel art.

`/help` lists everything. `/shortcuts` lists the key bindings.

## In a script

```bash
monarch -p "summarise the uncommitted changes"
git diff | monarch -p "review this"
```

`-p` runs a single turn, prints the answer on stdout and nothing else, and
exits non-zero when it could not run. It defaults to Default mode rather than
Auto — a script has not decided to stop being asked, a person has.

## Configuration

| Path | Holds |
| --- | --- |
| `~/.monarch/settings.json` | Provider, per-provider API keys and models, theme, trusted workspaces |
| `~/.monarch/persona.key` | Machine-local signing key for custom personas |
| `~/.monarch/memory.json` | What it remembers about you across sessions |
| `<project>/.monarch/settings.local.json` | Per-project personality, deny rules, hooks |
| `<project>/.monarch/attachments/` | Copies of files you attach |
| `MONARCH.md` | Project instructions loaded every turn |
| `AGENTS.md` | Read instead when a project has no `MONARCH.md` |

The older `~/.cymela` and `CYMELA.md` spellings are still read, and `CYMELA_*`
environment variables still work alongside `MONARCH_*`.

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
