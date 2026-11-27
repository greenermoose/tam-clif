# tam-clif roadmap

`tam-clif` is Tamlinux's pure-C17 terminal e-mail client built on
`libtam`, inspired by `aerc` and designed for both modern terminals and
bare Linux virtual consoles (`TERM=linux`).

- **Current version:** `0.0.0` (branch `dev`; planning only, no code yet)
- **Progress:** 0 of 7 phases done (P1 next)
- **Versioning:** Semantic versioning; pre-1.0 minor versions mark
  phases, culminating in `1.0.0` for daily-driver hardening.

## Phase summary

| Phase | Versions | Goal | Plan | Status |
| --- | --- | --- | --- | --- |
| **P1** | 0.1.0 | Build/sanitizer harness, INI config, and `tam_tty` multi-pane compositor | [P1-foundation.md](plans/P1-foundation.md) | Planned |
| **P2** | 0.2.0 | Maildir/Maildir++ scanner, RFC 5322 parser, message list viewport (MVP) | [P2-maildir-mvp.md](plans/P2-maildir-mvp.md) | Planned |
| **P3** | 0.3.0 | Recursive MIME parser, embedded pager, `[filters]` pipeline, attachments | [P3-mime-and-filters.md](plans/P3-mime-and-filters.md) | Planned |
| **P4** | 0.4.0 | JWZ threading, incremental search/filtering, and Maildir flag mutations | [P4-threading-and-search.md](plans/P4-threading-and-search.md) | Planned |
| **P5** | 0.5.0 | Compose/reply/forward templates, `$EDITOR` handoff, `msmtp` + `tam-ring` | [P5-compose-and-send.md](plans/P5-compose-and-send.md) | Planned |
| **P6** | 0.6.0 | Async background sync workers, `:` ex-command bar, `binds.ini`, multi-account tabs | [P6-async-and-commands.md](plans/P6-async-and-commands.md) | Planned |
| **P7** | 1.0.0 | PGP/MIME via `gpg`, git-patch workflow, read-only `mbox`, fuzzing and hardening | [P7-hardening.md](plans/P7-hardening.md) | Planned |

## Decided

- **Pure C17 and `libtam` only:** static link against `libtam.a`, zero
  external library dependencies beyond POSIX libc.
- **Process-based async and tool delegation:** delegate `mbsync`,
  `msmtp`/`sendmail`, `tam-ring`, `gpg`, and filters to controlled child
  processes so TLS and protocol parsers stay out of the TUI binary.
- **Never block the UI:** indexing, sync, and filters run off the main
  input/redraw loop.
- **Linux VT and modern terminal parity:** direct ANSI/xterm rendering
  via `tam_tty` with graceful `TERM=linux` fallback.

## Phases and milestones

### Phase P1: Foundation and terminal compositor (0.1.0) — [P1-foundation.md](plans/P1-foundation.md)

| Version | Milestone | Status |
| --- | --- | --- |
| **0.1.0** | Repo skeleton, C17 Makefile, ASan/UBSan test harness, `tam_ini` config, `tam_tty` multi-pane shell | Planned |

### Phase P2: Maildir engine and message list MVP (0.2.0) — [P2-maildir-mvp.md](plans/P2-maildir-mvp.md)

| Version | Milestone | Status |
| --- | --- | --- |
| **0.2.0** | Maildir/Maildir++ scanner, RFC 5322/2047 header parser, atomic flag updates, virtualized message list | Planned |

### Phase P3: MIME viewer and filter pipeline (0.3.0) — [P3-mime-and-filters.md](plans/P3-mime-and-filters.md)

| Version | Milestone | Status |
| --- | --- | --- |
| **0.3.0** | Recursive MIME parser, embedded pager, timeout-bounded `[filters]` pipeline, atomic attachment saving | Planned |

### Phase P4: Threading, search, and mailbox actions (0.4.0) — [P4-threading-and-search.md](plans/P4-threading-and-search.md)

| Version | Milestone | Status |
| --- | --- | --- |
| **0.4.0** | Client-side JWZ threading, incremental search and `:filter`, multi-select and undoable Maildir actions | Planned |

### Phase P5: Composition, templates, and sending (0.5.0) — [P5-compose-and-send.md](plans/P5-compose-and-send.md)

| Version | Milestone | Status |
| --- | --- | --- |
| **0.5.0** | Compose/reply/forward with templates, `$EDITOR` handoff, Drafts persistence, `msmtp` + `tam-ring` sending | Planned |

### Phase P6: Async workers, ex-commands, and keybinds (0.6.0) — [P6-async-and-commands.md](plans/P6-async-and-commands.md)

| Version | Milestone | Status |
| --- | --- | --- |
| **0.6.0** | Non-blocking `poll()` sync/filter workers, `:` ex-command bar with completion, context-aware `binds.ini` | Planned |

### Phase P7: PGP, patch workflows, and v1.0.0 hardening (1.0.0) — [P7-hardening.md](plans/P7-hardening.md)

| Version | Milestone | Status |
| --- | --- | --- |
| **1.0.0** | PGP/MIME via `gpg`, git-patch workflow (`:pipe`), read-only `mbox`, fuzz/sanitizer hardening, benchmarks | Planned |

## How a milestone is done

A milestone is done when its acceptance checks in the phase plan pass and
Fred has tried the build. When a milestone lands, bump `VERSION`, update
the phase plan's status, and update this roadmap in the same change.
