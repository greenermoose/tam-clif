# tam-clif Roadmap

This document outlines the architectural plan and phased roadmap for `tam-clif`:
a pure-C17, terminal e-mail client built on [`libtam`](https://github.com/greenermoose/libtam)
and designed to run natively on any Tamlinux system—including bare Linux virtual
consoles (`TERM=linux`).

**Current status:** Initial repository setup. No implementation exists yet.

---

## Architectural Principles

1. **Pure C17 & `libtam` Only:**
   - Static link against `libtam.a` (`tam_str`, `tam_fs`, `tam_fs_tree`,
     `tam_path`, `tam_exec`, `tam_ini`, `tam_tty`, `tam_key`, `tam_uni`).
   - Zero external library dependencies beyond POSIX libc.
2. **Process-Based Async & External Tool Delegation:**
   - Network sync (`mbsync`/`isync`), mail submission (`msmtp`/`sendmail`),
     OAuth2 tokens (`tam-ring`), PGP (`gpg`), and HTML/diff filters run as
     controlled child processes via `tam_exec` or non-blocking `pipe`/`poll`
     workers—keeping TLS and protocol parsers out of the TUI binary.
3. **Never Block the UI:**
   - Adopt `aerc`'s core lesson: Maildir indexing, background sync, and filter
     execution must never freeze keystrokes or screen redraws.
4. **Linux VT & Modern Terminal Parity:**
   - Direct ANSI/xterm rendering via `tam_tty` with graceful fallback on
     `TERM=linux` (8/16 colors, ASCII/UTF-8 box drawing) and synchronized frames
     on modern terminals.

---

## Release Versioning Plan

`tam-clif` follows [Semantic Versioning (SemVer 2.0.0)](https://semver.org/):

| Version | Phase | Milestone Scope | Status |
| :--- | :--- | :--- | :--- |
| **`v0.1.0`** | **Phase 0** | Repo skeleton, Makefile, ASan/UBSan test harness, `tam_tty` compositor, account/tab shell | Planned |
| **`v0.2.0`** | **Phase 1** | Maildir/Maildir++ scanner, RFC 5322 header parser, message list viewport & navigation (**MVP**) | Planned |
| **`v0.3.0`** | **Phase 2** | MIME multipart parser, embedded pager, `[filters]` pipe pipeline, attachment saving | Planned |
| **`v0.4.0`** | **Phase 3** | Client-side threading (`References`/`In-Reply-To`), search, filtering, Maildir flag mutations | Planned |
| **`v0.5.0`** | **Phase 4** | Message composer, reply/forward templates, `$EDITOR` handoff, `msmtp`/`sendmail` + `tam-ring` | Planned |
| **`v0.6.0`** | **Phase 5** | Async background sync worker, `:` ex-command bar, context-aware `binds.ini`, multi-account tabs | Planned |
| **`v1.0.0`** | **Phase 6** | PGP sign/verify/decrypt via `gpg`, git-patch workflow (`:pipe`), mbox read-only support, hardening | Planned |

---

## Phase 0: Foundation & Terminal Compositor (v0.1.0)

**Goal:** Establish the build system, sanitizer test suite, INI config loading,
and flicker-free terminal layout engine.

- `Makefile` with strict C17 flags (`-std=c17 -Wall -Wextra -Werror`) and
  `make test` under ASan/UBSan.
- Config loader using `tam_ini` and `tam_xdg_path` (`tam-clif.ini`,
  `accounts.ini`, `binds.ini`).
- Multi-pane screen compositor built on `tam_tty`, `tam_key`, and `tam_uni`:
  top tab bar (accounts), left folder list, main message list/viewer pane, and
  bottom status / `:` ex-command line.

## Phase 1: Maildir Engine & Message List MVP (v0.2.0)

**Goal:** Open and browse real local Maildir/Maildir++ mailboxes quickly.

- Maildir/Maildir++ discovery (`cur/`, `new/`, `tmp/`) via `tam_fs_walk` and
  `mmap` or buffered header scanning.
- Fast RFC 5322 / RFC 2047 header parser (`From`, `To`, `Subject`, `Date`,
  `Message-ID`, `In-Reply-To`, `References`, encoded-word UTF-8 decoding).
- Maildir flag parsing (`S` seen, `R` replied, `F` flagged, `T` trashed, `D`
  draft) and atomic flag-rename updates.
- Virtualized message-list viewport with Vim navigation (`j`/`k`, `g`/`G`,
  `Ctrl+d`/`Ctrl+u`) and folder switching.

## Phase 2: MIME Viewer & Filter Pipeline (v0.3.0)

**Goal:** Read complex multipart emails cleanly and safely.

- Recursive MIME parser (`multipart/mixed`, `multipart/alternative`,
  `text/plain`, `text/html`, `quoted-printable`, `base64`).
- Embedded scrollable message pager (`Enter` to open, `q` to close, `J`/`K` for
  next/prev message, `Ctrl+j`/`Ctrl+k` to cycle MIME parts).
- `aerc`-style `[filters]` pipeline using `tam_exec` (timeout-bounded): pipe
  `text/html` through `w3m`/`lynx`, `text/patch` through a diff colorizer, etc.
- Attachment listing and atomic extraction via `tam_fs_write_atomic`.

## Phase 3: Threading, Search & Mailbox Actions (v0.4.0)

**Goal:** Manage high-volume inboxes and mailing lists effortlessly.

- Client-side JWZ-style threading (`Message-ID` / `References` / `In-Reply-To`
  with optional `Subject` fallback) and collapsible thread trees.
- Incremental search (`/`, `?`, `n`, `N`) and `:filter` across headers/body.
- Multi-message selection/marking (`Space`, `v`) and bulk actions: mark
  read/unread, flag, archive, move, delete/trash, and undoable Maildir moves.

## Phase 4: Composition, Templates & Sending (v0.5.0)

**Goal:** Full send/reply/forward loop integrated with `$EDITOR`, `msmtp`, and
`tam-ring`.

- Compose new (`C`), reply (`r`), reply-all (`R`), forward (`f`), and edit draft.
- Lightweight template substitution (`{{From}}`, `{{Date}}`, `{{Subject}}`,
  quoted body) in `tam-clif` without heavy template engines.
- Handoff to `$EDITOR` (suspending `tam_tty` raw mode cleanly and restoring on
  exit), attachment picker, and Drafts Maildir persistence.
- Outgoing submission via `tam_exec` (`msmtp`, `sendmail`, or custom command)
  paired with `tam-ring access <account>` for OAuth2 credentials.

## Phase 5: Async Workers, Ex-Commands & Custom Keybinds (v0.6.0)

**Goal:** Make `tam-clif` responsive under background sync and fully
scriptable.

- Non-blocking `poll()` event loop integrating `tam_tty` input with child
  process stdout/stderr pipes and completion status (background `mbsync` sync,
  periodic refresh, async filters).
- `:` ex-command parser with tab-completion and history (`:check-mail`, `:cf`,
  `:filter`, `:pipe`, `:save`, `:compose`, `:next-account`, `:prev-account`).
- Context-aware keybinding engine loaded from `binds.ini` (`[default]`,
  `[messages]`, `[view]`, `[compose]`).

## Phase 6: PGP, Patch Workflows & v1.0.0 Hardening (v1.0.0)

**Goal:** Production-ready daily driver for Tamlinux.

- PGP/MIME signature verification, decryption, and signing via `tam_exec`
  calling `gpg`.
- First-class git patch workflow: `:pipe git am`, `:pipe git apply --check`,
  and quick diff inspection.
- Read-only `mbox` viewer support for archives.
- Fuzz/sanitizer coverage on RFC 5322/MIME/Maildir parsers and published
  cold-start / large-Maildir benchmarks.
