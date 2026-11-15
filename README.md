# tam-mail

A terminal e-mail client (MUA) written in C11, depending only on the standard C
libraries (POSIX libc) and [`libtam`](https://github.com/greenermoose/libtam).
Inspired by [aerc](https://git.sr.ht/~rjarry/aerc), `tam-mail` brings
asynchronous, keyboard-driven, multi-account email reading and composition to
Tamlinux with sub-megabyte memory overhead and first-class Linux virtual
console (`TERM=linux`) support.

> **Status:** Planning & initial repository setup (pre-`v0.1.0`). No C source
> files have been written yet; see [ROADMAP.md](ROADMAP.md) for the phased
> design and implementation plan.

## Design Philosophy

- **Zero Bloat, Pure C11 + `libtam`:** Links statically against `libtam.a`
  (`tam_str`, `tam_fs`, `tam_path`, `tam_exec`, `tam_ini`, `tam_tty`, `tam_key`,
  `tam_uni`). No Go runtime, no Python, no ncurses, no GLib, and no OpenSSL
  build dependency in the core binary.
- **Compose with Unix Mail Tools:** Rather than embedding a full TLS/IMAP/SMTP
  stack in the TUI binary, `tam-mail` reads local Maildir/Maildir++ stores
  directly and delegates network sync, sending, OAuth2, and MIME filtering via
  `tam_exec` to focused system tools (such as `mbsync`/`isync`, `msmtp`,
  `sendmail`, [`tam-ring`](../tam-ring), `gpg`, and `w3m`).
- **Asynchronous UI:** Inspired by `aerc`'s worker model, slow I/O (directory
  scans, sync subprocesses, MIME filter pipelines, and mail sending) runs
  asynchronously off the UI loop so navigation never freezes.
- **First-Class Linux VT & Modern Terminal Support:** Built on `tam_tty`,
  `tam_key`, and `tam_uni` for clean rendering on bare Linux VTs (`TERM=linux`,
  8/16 colors) as well as modern terminal emulators (`foot`, synchronized
  frames, 256/truecolor).

## Key Ideas Adopted from `aerc`

- **Tabbed Multi-Account Workspace:** Isolated per-account configuration in
  `accounts.ini` with independent tabs for personal, work, and mailing-list
  contexts.
- **Vim-Like Modal Ergonomics & `:` Ex-Command Line:** Context-aware bindings
  (`[default]`, `[messages]`, `[view]`, `[compose]`) configured via `binds.ini`,
  paired with an interactive `:` command bar (`:filter`, `:query`, `:pipe`,
  `:save`, `:compose`, `:cf`, `:quit`).
- **External Filter Pipeline:** Match MIME types or headers in `[filters]`
  (e.g., `text/html`, `text/patch`, `application/pgp-signature`) and pipe parts
  through external formatters before display.
- **Client-Side Threading:** Fast `References` / `In-Reply-To` / `Subject`
  threading over local Maildir indexes with collapsible threads.
- **Template-Driven Composition & `$EDITOR`:** Generate reply/forward headers
  and quoted bodies from templates, edit in `$EDITOR` (such as `tam-edit` or
  `vi`), and review/attach parts before sending.
- **First-Class Patch & Mailing-List Workflows:** Pipe patches directly to
  `git am` or `git apply`, colorize diffs, and navigate threads cleanly.

## Planned Configuration & Paths

Configuration and state follow XDG conventions via `tam_xdg_path`:

- `$XDG_CONFIG_HOME/tamlinux/tam-mail/tam-mail.ini` — UI, threading, and
  `[filters]` rules
- `$XDG_CONFIG_HOME/tamlinux/tam-mail/accounts.ini` — account definitions
  (Maildir paths, `sync_cmd`, `outgoing_cmd`, `oauth_cmd`)
- `$XDG_CONFIG_HOME/tamlinux/tam-mail/binds.ini` — context-sensitive keybindings

## Build Prerequisites (Planned)

- C11 compiler (`gcc` or `clang`)
- POSIX libc (glibc or musl) and GNU Make
- `libtam` (`libtam.a`)

See [ROADMAP.md](ROADMAP.md) for the milestone breakdown from `v0.1.0` to
`v1.0.0`.
