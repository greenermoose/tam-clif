# P2: Maildir engine and message list MVP

- **Status:** not started (target version 0.2.0)
- **Versions:** 0.2.0

## Goal

Open and browse real local Maildir/Maildir++ mailboxes quickly.

## Versions

- **0.2.0:**
  - Maildir/Maildir++ discovery (`cur/`, `new/`, `tmp/`) via
    `tam_fs_walk` and buffered/`mmap` header scanning.
  - Fast RFC 5322 / RFC 2047 header parser (`From`, `To`, `Subject`,
    `Date`, `Message-ID`, `In-Reply-To`, `References`, encoded-word UTF-8
    decoding).
  - Maildir flag parsing (`S`, `R`, `F`, `T`, `D`) and atomic
    flag-rename updates.
  - Virtualized message-list viewport with navigation and folder
    switching.

## Acceptance

- Unit and fixture tests against synthetic Maildir stores pass under
  ASan/UBSan.

## Dependencies

`libtam` (static) and POSIX libc only.

## Risks

- Malformed RFC 5322 headers or non-UTF-8 charsets; fuzz header parser
  with synthetic corpora.

## Open points

- None.
