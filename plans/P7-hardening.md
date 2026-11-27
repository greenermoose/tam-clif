# P7: PGP, patch workflows, and v1.0.0 hardening

- **Status:** not started (target version 1.0.0)
- **Versions:** 1.0.0

## Goal

Production-ready daily-driver terminal mail client for Tamlinux.

## Versions

- **1.0.0:**
  - PGP/MIME signature verification, decryption, and signing via
    `tam_exec` calling `gpg`.
  - First-class git patch workflow (`:pipe git am`,
    `:pipe git apply --check`, quick diff inspection).
  - Read-only `mbox` viewer support for archives.
  - Fuzz/sanitizer coverage on RFC 5322, MIME, and Maildir parsers plus
    published cold-start and large-Maildir benchmarks.

## Acceptance

- Full unit, fuzz, and sanitizer suites pass; benchmarks recorded; Fred
  accepts daily use.

## Dependencies

`libtam` (static); optional `gpg` and `git` invoked via `tam_exec`.

## Risks

- Untrusted PGP/MIME payloads; isolate `gpg` invocation and never pass
  passphrases on argv.

## Open points

- None.
