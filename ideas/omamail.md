# Ideas from omamail

*Evaluation notes from inspecting upstream `README.md`, `Cargo.toml`,
`docs/ARCHITECTURE.md`, `docs/SECURITY-BOUNDARIES.md`,
`docs/MESSAGE-RENDERING.md`, and repository layout (`src/`, `ui/`, `tests/`,
`benchmarks/`) at <https://github.com/huacnlee/omamail>.*

## What it is

- **Language:** Rust (edition 2024, `v0.11.3`, 213 files under `src/`) for
  the backend daemon/RPC worker, QML / Qt 6 (`ui/`, 430 files) for the
  shared UI, and a C++ / CMake standalone host (`app/`).
- **License:** MIT License.
- **Size:** 966 files total, including 78 test files and 37 benchmark files.
- **Scope:** A native mail and calendar client running either as an
  Omarchy 4 shell plugin (with bar widget, `mailto:` handling, and AI
  assistance) or as a standalone Qt desktop app. Supports Gmail API,
  Microsoft 365 / Outlook (OAuth), HEY (via `hey-cli`), JMAP, and
  IMAP/SMTP.

---

## Ideas worth taking

### 1. Strict Security Boundaries for Untrusted Mail Bytes — `adopt`
- `docs/SECURITY-BOUNDARIES.md` traces attacker-controlled bytes (mail
  headers/bodies, MIME filenames, server responses, URLs, persisted
  settings) through decoding and normalization to every final consumer.
- Specifically relevant to `tam-clif` (Phases 1–2 & 4):
  - Strip CR, LF, and NUL from outgoing header values and validate
    filenames/paths before atomic writes.
  - Keep credentials and OAuth tokens out of `argv`, logs, and
    world-readable config; pass secrets over stdin or `tam-ring`.
  - Enforce explicit deadlines and size bounds on every external process
    (`mbsync`, `msmtp`, `[filters]`).
  - Treat sender-written labels/subjects as untrusted plain text.

### 2. Bounded Parsing & Raw-Source Caching — `adopt`
- `docs/MESSAGE-RENDERING.md` explains why regex tag stripping fails on
  quoted `>` characters, enforces `MAX_TREE_DEPTH` on recursive walks, and
  caches raw sender HTML rather than sanitized output so parser fixes
  apply retroactively to already-cached mail.
- While `tam-clif` delegates `text/html` rendering to external `[filters]`
  (`w3m`/`lynx`), any built-in MIME tree walker, RFC 5322/2047 decoder,
  or fallback plain-text extractor in `tam-clif` should enforce strict
  recursion depth and size limits, and avoid caching derived render output
  as authoritative.

### 3. Action-Oriented Keyboard Architecture & Context Guards — `adopt`
- `docs/ARCHITECTURE.md` separates intent-based actions (`archive`,
  `reply`, `compose`) from input bindings (`ui/keys/Keymap.js` +
  `ui/components/KeyRouter.qml`), using the active context as the guard
  rather than duplicating `enabled` checks across handlers.
- This pairs cleanly with `aerc`'s `[default]` / `[messages]` / `[view]` /
  `[compose]` contexts in `tam-clif`'s `binds.ini` (Phase 5) and keeps
  keybindings and `:` ex-commands routed through one dispatch table.

### 4. Explicit State Ownership & Stable Identity — `adopt`
- `docs/ARCHITECTURE.md` assigns every mutable value one authoritative
  owner by lifetime (installation/account, across openings, current
  window, open popup/prompt) and requires keying list/pane state by
  stable message/folder identity rather than display row index.
- Directly applicable to `tam-clif` Phases 1 & 5: cursor position,
  marked-message selection (`Space`/`v`), and thread collapse state must
  key off stable Maildir filenames or `Message-ID`s so background `mbsync`
  rescans do not silently shift selections.

### 5. Corpus-Driven Mail Benchmarks & Adversarial Attachment Fixtures — `adopt`
- `omamail` maintains `benchmarks/mail/` (measuring load, pipeline, and
  concurrency) and `tests/fixtures/attachments/` (CRLF edge cases,
  charset edge cases, hidden/opaque PDF actions).
- Directly supports `tam-clif` `ROADMAP.md` Phase 6 goals: published
  cold-start and large-Maildir benchmarks plus fuzz/sanitizer coverage on
  RFC 5322, MIME, and Maildir parsers.

### 6. RTL / Bidi Subject Direction Handling — `consider`
- `docs/MESSAGE-RENDERING.md` notes that Latin reply prefixes (`Re:`,
  `Fwd:`) can override Arabic/Persian subject direction if resolved
  naively (`Direction.resolveSubject`).
- Worth evaluating for `tam-clif` (via `tam_uni`) when rendering mixed-direction
  subjects in modern terminals, while preserving clean `TERM=linux`
  fallback.

### 7. Provider Capability Model — `consider`
- `docs/ARCHITECTURE.md` models capabilities per provider so unsupported
  actions are omitted from the view rather than failing after activation.
- Useful if `tam-clif` distinguishes full Maildir accounts from read-only
  `mbox` archives (Phase 6) or send-disabled accounts.

### 8. Built-in Calendar, Desktop Bar Widget & AI Draft Assistant — `reject`
- Out of scope for `tam-clif`. A pure C17 terminal MUA with `TERM=linux`
  parity should stay focused on reading, searching, composing, and patch
  workflows; calendar, bar widgets, and AI belongs in separate tools.

---

## Things to avoid

- **Heavy Multi-Language Runtime Stack:** Embedding Rust + `tokio` +
  `rusqlite` + `reqwest` + `rustls` + Qt 6 / QML + C++ host violates
  `tam-clif`'s `libc` + `libtam.a`-only constraint.
- **In-Process Network, TLS & OAuth2 Protocols:** `omamail` implements
  Gmail API, Microsoft Graph, JMAP, IMAP, SMTP, DNS resolution, and
  keyring DBus/Keychain bindings inside its backend binary. `tam-clif`
  delegates sync, send, and OAuth2 tokens to `mbsync`, `msmtp`, and
  `tam-ring` via `tam_exec`.
- **GUI / Qt Rich-Text & Image-Downscaling Pipeline:** Inline `cid:` image
  decoding/downscaling, QML `XMLHttpRequest` timer workarounds, and Qt
  rich-text sanitizer quirks are unnecessary in a terminal UI.
- **Embedded AI Agent Orchestration:** Coupling external AI CLIs
  (Claude/Codex/OpenCode) and chat state into the mail client adds
  complexity and non-Unix coupling.
- **Omarchy Coupling:** `omamail` targets Omarchy 4 shell APIs, themes,
  and `bindings.lua`. `tam-clif` is standalone Tamlinux software with zero
  Omarchy dependencies.

---

## Where to look

- `docs/SECURITY-BOUNDARIES.md` — untrusted mail-byte tracing, header/URL
  validation, credential isolation
- `docs/MESSAGE-RENDERING.md` — `MAX_TREE_DEPTH`, raw-source caching,
  `Direction.resolveSubject`, MIME boundary generation
- `docs/ARCHITECTURE.md` — state ownership lifetimes, stable identity,
  intent-based action/context separation
- `docs/KEYS.md` and `ui/keys/Keymap.js` — declarative action-to-context
  keymaps
- `tests/fixtures/attachments/` and `benchmarks/mail/` — adversarial
  fixtures and mail benchmark harness design
