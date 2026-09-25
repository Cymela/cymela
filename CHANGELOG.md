# Changelog

Monarch CLI ships as a compiled bundle on npm. This file records what changed
in each published version. Versions up to 0.1.5 were published as `cymela`.

## 2.0.3

Mostly about what each request costs and whether it succeeds at all, plus a
chameleon that moves.

### Fixed

- **The chameleon moves while you can see it.** Since 2.0.0 it only moved when
  something else redrew the screen, so it sat still unless you were typing or a
  reply was streaming. It now animates whenever it is on screen, and stops when
  you scroll it out of view.
- **Requests stay cacheable for the whole conversation.** The short description
  of your repository in the system prompt was taken again every turn. After the
  model edited a file its change count moved, and every provider that caches by
  prefix (DeepSeek, OpenAI, Anthropic, Gemini) billed the rest of the
  conversation at the full price on the next request. It is now taken once per
  conversation. Measured after an edit: the next request went from 61% to
  99.9% cacheable.
- **Claude through the Anthropic provider.** The default model was
  `claude-sonnet-5-20260630`, which is not an id Anthropic serves; it is now
  `claude-sonnet-5`, and the old id is corrected if you have it saved. Current
  Claude models (Sonnet 5, Opus 5 and 5.5, Opus 4.7 and 4.8, Fable) were sent a
  thinking setting they reject, so each request failed once and was retried
  without thinking, and `/effort` never reached them. They now get adaptive
  thinking at the effort you chose. Requests also ask Anthropic to cache the
  prompt, which makes cached input a tenth of the price, and on Opus 5.5 and
  Fable 5.1 they ask for outdated thinking to be dropped rather than refused,
  so switching mode or compacting mid-conversation no longer fails the next
  request on newer Anthropic accounts.
- **DeepSeek effort levels reach DeepSeek.** `/effort low` used to think as
  hard as the default, and `max` never asked for DeepSeek's own maximum. Both
  now send DeepSeek's `reasoning_effort`.
- **`/compact` compacts.** On million-token models it answered "nothing worth
  compacting" even at 170k tokens. Manual compaction now works from 80k, and
  when it does decline, pressing C within ten seconds compacts anyway.
- **Math prints as readable text.** LaTeX in replies, such as `\frac{a}{b}` or
  `x^{2}`, is shown as a/b and x² instead of raw backslashes.
- **A clean repository reads clean.** Monarch's own `.monarch/` folder was
  counted as an uncommitted change, so every checkout showed "1 file changed"
  after the first launch.
- **A model with a smaller output limit no longer fails at max effort.**
  Claude Haiku 4.5 stops at 64,000 output tokens and the max tier asked for
  65,536. The limit is now read from the provider's refusal and the request is
  sent again within it.

### Changed

- **DeepSeek defaults to `deepseek-flash`**, DeepSeek's current model, which
  can see images. Images you attach now go straight to it; before, a DeepSeek
  model was only told where the file was saved. The unused remains of 0.1.5's
  route for having a second model describe an image are removed, including the
  `CYMELA_VISION_API_KEY` name.
- **The model is told today's date and what kind of machine it is on**, so
  searches use the current year and a Linux machine is not handed Windows
  commands. What is sent to your provider: the date, the operating system and
  its version, processor architecture, core count, memory and Node version.
  No hostname, username or home folder.
- **`AGENTS.md` is read** when a project has no `MONARCH.md` (or `CYMELA.md`,
  `HYPER.md`), so a repository set up for Codex, Gemini CLI, Cursor, Copilot or
  Claude Code works here without a second file.
- **Claude models through OpenRouter are asked to cache the prompt**, the same
  as on Anthropic's own API.
- **Cost estimates use current prices** for DeepSeek and Claude. They are still
  estimates at list price: cache discounts are not counted yet.

## 2.0.2

Two OpenRouter changes: one fix, and one new behaviour that is disclosed here
rather than slipped in.

### Fixed

- **OpenRouter works again on every effort tier.** Requests asked for
  reasoning with both an effort level and a token budget, which OpenRouter
  accepts one of but not both. Every tier above `low`, including the default,
  was rejected with a 400 before a single token came back. This was present in
  2.0.0 and 2.0.1. The request now sends the effort tier alone and lets
  OpenRouter translate it for whichever model is behind it. `/effort low` was
  the only setting that worked; it no longer needs to be.

### Changed

- **Requests to OpenRouter now identify the app as Monarch CLI.** Three
  headers: the app's page (`https://cymela.com/cli`), its name, and the
  category `cli-agent`. This is what gives the app a page on OpenRouter with
  per-model usage analytics and makes it eligible for OpenRouter's public app
  rankings. The headers go to OpenRouter only, carry a name and a URL and
  nothing about you, and nothing is sent to Cymela. It does mean your
  OpenRouter usage through this tool counts toward the app's public numbers.
  "Runs entirely on your machine" is unchanged.

## 2.0.1

Two fixes, no new behaviour.

### Fixed

- **A hook that times out no longer leaves its work running.** Hooks run
  through a shell, and on a timeout only the shell was being signalled, so
  anything the hook had started kept going without it. A `PreToolUse` hook runs
  on every tool call, so a hook that hung left one more process behind each
  time, for as long as the session lasted. The timeout now takes down the whole
  process group.
- **The agent is told where a session message actually goes.** Sending to
  another session is a local file, and 2.0.0 said so. But the receiving session
  reads the message into its context, and that context goes to whichever
  provider *that* session is configured with. The model doing the sending is
  the only one positioned to judge what belongs in a message, and it was not
  being told this. It is now, along with an instruction never to send a key, a
  token or a credential.

## 2.0.0

The CLI formerly published as `cymela` is now **Monarch CLI**. Both
`npm install -g monarchai` and `npm install -g cymela` install it, and the
command is `monarch` either way.

This is a rebuild rather than an increment, which is why the number jumps.

### Upgrading

- **`npm install -g cymela` replaces the old install** and keeps the `cymela`
  command working, now pointing at the current build.
- **Your settings move with you.** Provider, per-provider API keys, model
  choices, theme and custom personas are carried from `~/.cymela` to
  `~/.monarch` on first run, once. Custom personas keep their machine signing
  key, so they still verify rather than being dropped. Conversations were
  already stored per project and are read where they are.
- **If both npm names end up installed**, `monarch --doctor` reports it and
  gives you the one line that clears it up.

### New

- **Sub-agents with roles.** Scout (read-only), Builder and Checker run under
  the lead agent with their own tool limits, streamed live into `/agents`.
- **Cross-session messaging.** Two sessions running on the same machine can
  send each other messages, including across different projects. Delivery is to
  a running session; if the other side is not running, the send fails and says
  so rather than queuing. The agent sends these itself once you have asked it
  to keep another session posted, including turns later without being asked
  again.

  Delivery is local: the message is a file in your own config directory, and
  no server or network is involved in moving it. What happens next is not
  local. The receiving session reads the message into its context, and that
  context goes to whichever model provider *that* session is configured with,
  the same as everything else it reads. So a message you send to another
  session reaches that session's provider. If the two sessions are pointed at
  different providers, it reaches the other one.
- **Memory across sessions.** Durable facts you ask it to keep, stored only on
  your machine. `/memory` shows everything, `/memory off` stops it collecting.
- **Scriptable.** `monarch -p "…"` runs one turn with no UI, and now reads
  piped input, so `git diff | monarch -p "review this"` works.
- **Project awareness.** Each session starts already knowing the project, its
  language, the branch, whether the tree is dirty and how the project is tested,
  instead of spending its first few calls finding out.
- **Two more providers**, bringing it to thirteen: NVIDIA NIM and Microsoft
  Foundry (Azure).
- **`monarch --doctor`** checks Node, terminal, provider, key, key-file
  permissions, config writability and store integrity, and every failure comes
  with the command that fixes it.

### Changed

- **`/execute`, `/agent` and `/override` are retired** in favour of
  `/run-plan`, `/default` and `/auto`. Typing an old one tells you its
  replacement. `/mode` covers all three from one command.
- **Project instructions are `MONARCH.md`.** `CYMELA.md` and `HYPER.md` are
  still read.
- **Environment variables are `MONARCH_*`.** `CYMELA_*` and `HYPER_*` still
  work.
- **Config lives in `~/.monarch`.** `~/.cymela` is still read.

### Platform

2.0.0 was developed and tested on Linux, which is a change from 0.1.x. It is
written to be portable and the platform-specific paths are covered by tests,
but it has not had real hours on Windows or macOS yet. Reports from either are
genuinely useful — run `monarch --doctor` first and include its output.

### Requires

Node.js 20 or newer. Nothing is compiled on install and there are no runtime
dependencies.

## 0.1.4

- **Fixed: shell commands containing double quotes ran mangled on Windows.**
  The spawn layer re-quoted them with an escape style cmd.exe does not parse,
  so `powershell -Command "…"` printed its own command text instead of
  executing it (silently dropping `$_` on the way), and a quoted URL reached
  curl broken ("URL rejected"). Commands are now handed to cmd verbatim,
  exactly as typed. Both the pty and pipe backends were affected, on every
  Windows machine.
- **Fixed: the screen could freeze during long thinking streams** (Ctrl+T
  live thoughts) on slow terminal hosts — most visibly a maximized classic
  Windows console. A bottom-pinned transcript shifts every row on each
  streamed chunk, and the renderer was rewriting the whole screen ~5×/second
  (measured at 80–120 KB/s on a 100-row window). It now emits one scroll and
  paints only the new rows — a ~10× reduction — and when a terminal still
  falls behind, stale intermediate frames are skipped so it always shows the
  newest one instead of replaying the backlog.
- README: corrected the description of **Hyper** — it is Cymela's default
  persona (a tone and style preset), not the engine. The CLI runs on whichever
  model provider you configure.

## 0.1.3

- **Fixed: plain Enter did not submit the prompt.** A heuristic in the
  terminal input library flagged every Enter as Shift+Enter, so the composer
  inserted a newline instead of sending — in every terminal, on every
  platform. Versions 0.1.0–0.1.2 are all affected; if you installed one of
  them, update. The fix is covered by a new test harness that drives the real
  TUI with no pseudo-terminal in the middle, so this class of bug can no
  longer hide from CI.
- The provider list now always opens with the cursor on the first entry; the
  previously used provider keeps its "last used" tag but no longer pre-claims
  the cursor.

## 0.1.2

- Removed the post-install greeting. npm discards lifecycle-script output, so
  the message never reached anyone; the package now ships with no install
  scripts of its own.

## 0.1.1

First release with Linux and macOS treated as fully supported rather than
best-effort. Everything here was found by running Cymela on those platforms.

- Fixed a crash on Linux and macOS where a hook that exited without reading its
  input took the whole agent down with it
- Terminal state — alternate screen, cursor, mouse mode, bracketed paste — is
  now restored on every exit path, including crashes and signals
- `SIGTERM`, `SIGHUP` and `Ctrl+Z` behave correctly on Linux and macOS
- Clipboard paste-attach and image resizing now work on macOS (`pbpaste`,
  `sips`) and Linux (`wl-paste`, `xclip`, ImageMagick)
- `sudo` and `doas` are recognised as privileged commands and prompt separately
- Shell commands run under `bash` or `sh` on POSIX, and the transcript labels
  the shell you are actually using
- Provider errors are reported in plain English instead of raw status codes and
  JSON — rate limits, bad keys and billing problems each say what to do
- Arrow keys and Home/End no longer get swallowed when pressed immediately
  after pasting
- `node-pty` is now an optional dependency; if the native build is unavailable,
  Cymela falls back to pipe-based shells instead of failing to install

## 0.1.0

First public release.
