# P4: Threading, search, and mailbox actions

- **Status:** not started (target version 0.4.0)
- **Versions:** 0.4.0

## Goal

Manage high-volume inboxes and mailing lists effortlessly.

## Versions

- **0.4.0:**
  - Client-side JWZ-style threading (`Message-ID` / `References` /
    `In-Reply-To` with optional `Subject` fallback) and collapsible
    thread trees.
  - Incremental search (`/`, `?`, `n`, `N`) and `:filter` across headers
    and body.
  - Multi-message marking and bulk actions: read/unread, flag, archive,
    move, trash, and undoable Maildir moves.

## Acceptance

- Threading, search, and bulk Maildir mutations pass synthetic fixture
  tests under ASan/UBSan.

## Dependencies

`libtam` (static) and POSIX libc only.

## Risks

- Cycles or broken references in `References` headers; enforce cycle
  detection in JWZ threading.

## Open points

- None.
