# P1: Foundation and terminal compositor

- **Status:** not started (target version 0.1.0)
- **Versions:** 0.1.0

## Goal

Establish the build system, sanitizer test suite, INI config loading, and
flicker-free multi-pane terminal layout engine.

## Versions

- **0.1.0:**
  - `Makefile` with strict C17 flags (`-std=c17 -Wall -Wextra -Werror`)
    and `make test` under ASan/UBSan.
  - Config loader using `tam_ini` and `tam_xdg_path` (`tam-clif.ini`,
    `accounts.ini`, `binds.ini`).
  - Multi-pane screen compositor built on `tam_tty`, `tam_key`, and
    `tam_uni`: top account tab bar, left folder list, main message
    list/viewer pane, and bottom status / `:` ex-command line.

## Acceptance

- `make` and `make test` pass cleanly under ASan/UBSan.
- `tam-clif --version` prints `VERSION`; `readelf -d` lists libc only.

## Dependencies

`libtam` (static) and POSIX libc only.

## Risks

- Layout differences between `TERM=linux` and modern Wayland terminals;
  test both 8/16-color ASCII fallback and UTF-8 box drawing early.

## Open points

- Confirm initial default `tam-clif.ini` schema with Fred before coding.
