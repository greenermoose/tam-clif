# `aerc` Architecture & `tam-clif` Adaptation Notes

`tam-clif` is currently in pre-`v0.1.0` planning (`ROADMAP.md`), with no C source files yet, and no `aerc` Go source tree is present locally. This document summarizes the architectural mechanisms and algorithms described in `ideas/aerc.md` alongside `tam-clif`'s planned C11/`libtam` design in `ROADMAP.md`.

---

## 1. Concurrency & Worker Model

### `aerc` (`ideas/aerc.md`)
- Uses **in-process Go goroutine workers** for native IMAP, JMAP, SMTP, Notmuch, and Maildir backends, keeping slow network I/O, directory scans, and MIME filters off the UI thread.

### Planned `tam-clif` Adaptation (`ROADMAP.md`)
- **Local-first Maildir/Maildir++ engine + `poll()` event loop (`v0.2.0`, `v0.6.0`)**:
  - Reads local Maildir (`cur/`, `new/`, `tmp/`) directly via `tam_fs_walk` with `mmap` or buffered header scanning.
  - Delegates network sync (`mbsync`/`isync`), SMTP submission (`msmtp`/`sendmail`), OAuth2 (`tam-ring access <account>`), and PGP (`gpg`) to child processes via `tam_exec` or non-blocking `pipe`/`poll()` workers multiplexed alongside `tam_tty` input.

---

## 2. Maildir Storage, Header Parsing & Threading

### `aerc` (`ideas/aerc.md`)
- Performs client-side threading over local Maildir stores using `References`, `In-Reply-To`, and `Subject`.

### Planned `tam-clif` Data Structures & Algorithms (`ROADMAP.md`)
- **Maildir Flag State & Atomic Renames (`v0.2.0`)**: Parses Maildir filename flags (`S`, `R`, `F`, `T`, `D`) and performs atomic `rename()` updates (with undoable move tracking in `v0.4.0`).
- **Fast RFC 5322 / RFC 2047 Header Scanner (`v0.2.0`)**: Extracts `From`, `To`, `Subject`, `Date`, `Message-ID`, `In-Reply-To`, and `References`, decoding RFC 2047 encoded-words to UTF-8.
- **JWZ-Style Threading (`v0.4.0`)**: Client-side JWZ-style thread tree construction keyed by `Message-ID`, `References`, and `In-Reply-To` (with optional `Subject` fallback) and collapsible tree state for the virtualized message-list viewport.
- **Incremental Search & Filter (`v0.4.0`)**: `/` / `?` / `n` / `N` search and `:filter` across headers and bodies, plus multi-message marking (`Space`, `v`).

---

## 3. Recursive MIME Parser & External Filter Pipeline

### `aerc` (`ideas/aerc.md`)
- Parses MIME trees and pipes specific content types (`text/html`, `text/patch`, etc.) through external CLI `[filters]` (`w3m`, `lynx`, diff colorizers) rather than embedding an HTML engine.

### Planned `tam-clif` Adaptation (`ROADMAP.md`, `v0.3.0`)
- **Recursive MIME Tree Parser**: Handles `multipart/mixed`, `multipart/alternative`, `text/plain`, `text/html`, `quoted-printable`, and `base64` decoding.
- **Timeout-Bounded `[filters]` Pipe Pipeline**: Streams MIME part payloads through external filter commands via `tam_exec` with timeout bounds into an embedded scrollable pager (`J`/`K`, `Ctrl+j`/`Ctrl+k` to cycle parts).
- **Atomic Attachment Extraction**: Writes extracted attachments via `tam_fs_write_atomic`.

---

## 4. Terminal UI, Modal Input & Editor Handoff

### `aerc` (`ideas/aerc.md`)
- Uses `tcell` for a tabbed multi-account UI, context-aware modal keybindings (`[default]`, `[messages]`, `[view]`, `[compose]`), a `:` ex-command bar with history and tab-completion, Go `text/template` for message templates, and an **embedded PTY terminal emulator** (`term`) to host `$EDITOR` inside a tab.

### Planned `tam-clif` Adaptation (`ROADMAP.md`, `v0.1.0`, `v0.5.0`, `v0.6.0`)
- **Multi-Pane Compositor (`tam_tty`, `tam_key`, `tam_uni`)**: Tabbed accounts, folder list, virtualized message list/viewer, and `:` ex-command bar with history and tab-completion—supporting both `TERM=linux` and modern terminals without `tcell`/`terminfo`.
- **Direct `$EDITOR` Handoff (Avoiding Embedded PTY)**: Suspends `tam_tty` raw mode, runs `$EDITOR`, and restores raw mode on exit.
- **Configuration & Templates**: `tam_ini` (`tam-clif.ini`, `accounts.ini`, `binds.ini`) and lightweight C `{{From}}`/`{{Date}}`/`{{Subject}}` template substitution.
