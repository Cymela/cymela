# Changelog

Cymela ships as a compiled bundle on npm. This file records what changed in
each published version.

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
