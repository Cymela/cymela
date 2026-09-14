# Changelog

Monarch CLI ships as a compiled bundle on npm. This file records what changed
in each published version. Versions up to 0.1.5 were published as `cymela`.

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
