# Ideas from aerc

## Overview

- **What it is:** An asynchronous, keyboard-driven terminal email client (MUA)
  with a Vim-like interface, tabbed multi-account workspace, and strong
  developer / mailing-list / git-patch workflows.
- **Developers:** Originally created by Drew DeVault; maintained by Robin Jarry
  (`rjarry`) and contributors.
- **License:** MIT License.
- **Language:** Go (using `tcell` for terminal UI and Go's standard/ecosystem
  network & mail libraries).
- **Source Code:** <https://git.sr.ht/~rjarry/aerc> (mirror:
  <https://github.com/rjarry/aerc>).

---

## Ideas We Borrow

### 1. Asynchronous UI Never Blocks on I/O
- `aerc`'s defining principle ("asynchronous email reading client") is that slow
  syncs, large directory scans, or MIME filters must never freeze keystrokes or
  screen redraws.
- We adopt this via a non-blocking `poll()` event loop pairing `tam_tty` input
  with child process pipes.

### 2. Vim-Like Modal Ergonomics & `:` Ex-Command Bar
- Single-key navigation (`j`/`k`, `g`/`G`, `Ctrl+u`/`Ctrl+d`, `J`/`K` next/prev
  message in viewer, `Ctrl+j`/`Ctrl+k` cycle MIME parts).
- Bottom `:` command line (`:filter`, `:query`, `:pipe`, `:save`, `:compose`,
  `:cf`, `:check-mail`, `:quit`) with history and tab-completion.

### 3. Context-Aware Keybindings
- Separate binding contexts (`[default]`, `[messages]`, `[view]`, `[compose]`)
  so keys behave naturally in each mode without collisions.

### 4. `[filters]` MIME & Header Pipeline
- Piping MIME parts (`text/html`, `text/patch`) through external CLI filters
  (`w3m`/`lynx`, diff colorizers) keeps rendering modular and avoids embedding
  an HTML engine.

### 5. Tabbed Multi-Account Workspace & Client-Side Threading
- Independent per-account tabs with isolated folders and commands.
- Client-side `References` / `In-Reply-To` / `Subject` threading over local
  Maildir stores.

### 6. First-Class Git Patch & Mailing-List Support
- `:pipe git am` / `:pipe git apply --check` and clean diff viewing for
  patch-based development.

---

## Ideas We Adapt

### 1. Worker / Backend Architecture
- **aerc:** Implements in-process Go goroutine workers for native IMAP, JMAP,
  SMTP, Notmuch, and Maildir.
- **tam-mail:** Keeps the core binary strictly `libc` + `libtam.a` by reading
  local Maildir/Maildir++ stores directly and delegating network sync/send/auth
  via `tam_exec` to `mbsync`/`isync`, `msmtp`/`sendmail`, `tam-ring`, and
  `gpg`.

### 2. Configuration & Templates
- **aerc:** Uses INI-like `aerc.conf`/`accounts.conf`/`binds.conf` plus Go's
  `text/template`.
- **tam-mail:** Uses `tam_ini` (`tam-mail.ini`, `accounts.ini`, `binds.ini`)
  and a lightweight C template substitution engine (`{{From}}`, `{{Date}}`,
  `{{Subject}}`) for replies/forwards.

---

## Ideas We Avoid

- **Embedded Go Runtime & In-Process TLS/IMAP/JMAP Stack:** Embedding full
  network protocols and TLS inside the TUI binary violates `tam-mail`'s
  `libc` + `libtam`-only constraint and duplicates what `mbsync`/`msmtp`
  already do well.
- **Embedded Terminal Emulator (`term`):** `aerc` embeds a full PTY terminal
  emulator for running `$EDITOR` and shells inside tabs. In `tam-mail`,
  suspending `tam_tty` raw mode to run `$EDITOR` (e.g., `tam-edit`) cleanly
  and restoring on exit is far simpler, smaller, and works identically on
  `TERM=linux`.
- **`tcell` / `terminfo` Dependency:** Replaced by `libtam`'s `tam_tty`,
  `tam_key`, and `tam_uni`.
