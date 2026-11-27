# P5: Composition, templates, and sending

- **Status:** not started (target version 0.5.0)
- **Versions:** 0.5.0

## Goal

Full compose/reply/forward loop integrated with `$EDITOR`, `msmtp`, and
`tam-ring`.

## Versions

- **0.5.0:**
  - Compose new, reply, reply-all, forward, and edit draft.
  - Lightweight template substitution (`{{From}}`, `{{Date}}`,
    `{{Subject}}`, quoted body).
  - Clean `$EDITOR` handoff (suspending and restoring `tam_tty` raw
    mode), attachment picker, and Drafts Maildir persistence.
  - Outgoing submission via `tam_exec` (`msmtp`/`sendmail`) paired with
    `tam-ring access <account>` for OAuth2 credentials.

## Acceptance

- End-to-end compose/reply/draft/send tests pass against a mock
  submission script.

## Dependencies

`libtam` (static); external `$EDITOR`, `msmtp`/`sendmail`, and
`tam-ring` invoked via `tam_exec`.

## Risks

- Accidental sends or lost drafts; always save drafts atomically before
  submission and never access Fred's live mail in tests.

## Open points

- None.
