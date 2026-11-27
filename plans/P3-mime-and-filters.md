# P3: MIME viewer and filter pipeline

- **Status:** not started (target version 0.3.0)
- **Versions:** 0.3.0

## Goal

Read complex multipart emails cleanly and safely.

## Versions

- **0.3.0:**
  - Recursive MIME parser (`multipart/mixed`, `multipart/alternative`,
    `text/plain`, `text/html`, `quoted-printable`, `base64`).
  - Embedded scrollable message pager with MIME part cycling.
  - `aerc`-style `[filters]` pipeline using timeout-bounded `tam_exec`
    (e.g., `text/html` through `w3m`/`lynx`, `text/patch` through a diff
    colorizer).
  - Attachment listing and atomic extraction via `tam_fs_write_atomic`.

## Acceptance

- Multipart/nested MIME fixtures render cleanly; filter timeouts do not
  freeze the UI; attachment extraction preserves safe filenames.

## Dependencies

`libtam` (static); optional external filter commands invoked via
`tam_exec`.

## Risks

- Deeply nested MIME or malicious attachment filenames; bound recursion
  depth and sanitize output paths.

## Open points

- None.
