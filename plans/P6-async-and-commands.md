# P6: Async workers, ex-commands, and custom keybinds

- **Status:** not started (target version 0.6.0)
- **Versions:** 0.6.0

## Goal

Make `tam-clif` responsive under background sync and fully scriptable.

## Versions

- **0.6.0:**
  - Non-blocking `poll()` event loop integrating `tam_tty` input with
    child process pipes (background `mbsync`, periodic refresh, async
    filters).
  - `:` ex-command parser with tab-completion and history (`:check-mail`,
    `:cf`, `:filter`, `:pipe`, `:save`, `:compose`, account switching).
  - Context-aware keybinding engine loaded from `binds.ini` (`[default]`,
    `[messages]`, `[view]`, `[compose]`).

## Acceptance

- Background workers never block keystrokes or redraws; ex-command and
  keybind tests pass under ASan/UBSan.

## Dependencies

`libtam` (static); external `mbsync` invoked via non-blocking child
process.

## Risks

- Child-process pipe deadlocks or zombie processes; handle `SIGCHLD` and
  bounded pipe buffers cleanly.

## Open points

- None.
