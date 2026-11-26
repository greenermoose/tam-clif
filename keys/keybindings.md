# tam-clif Keybindings (First Draft)

`tam-clif` follows the
[Tamlinux keybinding standard](https://github.com/greenermoose/tam-idol/blob/dev/keys/keybindings.md).
This document specifies the first-draft keybindings across `tam-clif`'s
contexts (`[default]`, `[messages]`, `[view]`, `[compose]`, and the
prompt / `:` ex-command bar), how universal `Ctrl` chords behave in
each context, and how every essential action works on both graphical
terminals (`tam-last`) and the Linux console (`TERM=linux`).

## Principles

- **Universal `Ctrl` map, zero collisions.** Every `Ctrl` chord keeps
  its single Tamlinux meaning. Actions that do not apply in a context
  (for example `Ctrl+R` Replace in `[messages]` or `[view]`) are no-ops
  rather than taking on an unrelated second meaning.
- **No `Meta` bindings.** `Meta` is owned exclusively by `tam-pane`.
- **No kernel VT or legacy ASCII collisions.** `Alt+Left`, `Alt+Right`,
  and `Alt+F1`–`Alt+F12` are never bound (captured by the Linux kernel VT
  driver on `TERM=linux`). `Ctrl+H` (`Backspace`), `Ctrl+I` (`Tab`),
  `Ctrl+J` / `Ctrl+M` (`Enter`), and `Ctrl+[` (`Escape`) are never
  assigned separately.
- **Three-physical-key ceiling.** No default chord uses more than two
  modifiers plus one primary key.
- **Physical key names and canonical widgets.** Documentation and
  `binds.ini` use physical key names (`Ctrl`, `Alt`, `Shift`, `Enter`,
  `Backspace`, `Delete`, `Tab`, `Escape`, `Space`, `Left`, `Right`,
  `Up`, `Down`, `Home`, `End`, `PageUp`, `PageDown`) and lowercase
  hyphenated widget names.
- **One default map, user-rebindable.** No built-in vi or readline
  modes; users override or unbind keys in `binds.ini`.

## Intentional departures from `aerc`

`tam-clif` adopts `aerc`'s context-aware modal ergonomics and `:`
ex-command line while replacing `aerc` bindings that conflict with the
Tamlinux keybinding standard:

| `aerc` binding | `aerc` action | Why it changes in `tam-clif` | `tam-clif` replacement |
| --- | --- | --- | --- |
| `Ctrl+U` | Half-page up | `Ctrl+U` is universal `update` | `PageUp` / `b` (`Ctrl+U` syncs mailbox) |
| `Ctrl+D` | Half-page down | `Ctrl+D` is universal `close` | `PageDown` / `Space` (`Ctrl+D` closes tab/pane) |
| `Ctrl+J` | Next MIME part | Collides with `Enter` (`0x0A`) on `TERM=linux` | `Alt+J` (alias `]`) |
| `Ctrl+K` | Prev MIME part | `Ctrl+K` is universal `kill-line` | `Alt+K` (alias `[`) |

## Universal `Ctrl` map

All 22 ASCII-clean `Ctrl+<letter>` slots plus `Ctrl+\` follow the
Tamlinux standard verbatim:

| Key | Widget | Behavior in `tam-clif` |
| --- | --- | --- |
| `Ctrl+A` | `select-all` | Select all messages in `[messages]`, or select all text in prompt/compose fields |
| `Ctrl+B` | *(reserved)* | No-op |
| `Ctrl+C` | `copy` | Copy selected text (viewer/prompt/compose), or selected message Subject/`Message-ID` (OSC 52 write + internal clipboard) |
| `Ctrl+D` | `close` | Close active account tab, `[view]` pane, or `[compose]` buffer (prompts if unsaved) |
| `Ctrl+E` | `execute` | Open named-command palette & one-handed chord composer (`Tab` cycles modifiers on `TERM=linux`) |
| `Ctrl+F` | `find` | Incremental search in `[messages]` list, `[view]` body, or folder sidebar |
| `Ctrl+G` | `guidance` | Toggle contextual keybinding guidance overlay |
| `Ctrl+K` | `kill-line` | Clear current prompt / `:` input line; no-op in read-only lists and `[view]` |
| `Ctrl+L` | `clear-screen` | Clear and redraw terminal screen |
| `Ctrl+N` | `new` | Compose a new message (opens `[compose]`) |
| `Ctrl+O` | `open` | Open selected message in `[view]`, or open folder/attachment switcher |
| `Ctrl+P` | `pause` | Suspend `tam-clif` (`SIGTSTP`), restoring `tam_tty` modes cleanly |
| `Ctrl+Q` | `quit` | Quit `tam-clif` (prompts if unsent `[compose]` buffers exist) |
| `Ctrl+R` | `replace` | Find-and-replace in editable fields; no-op in read-only views |
| `Ctrl+S` | `save` | Save part/message in `[view]`; save draft in `[compose]`; no-op in `[messages]` |
| `Ctrl+T` | *(reserved)* | No-op |
| `Ctrl+U` | `update` | Sync/refresh current mailbox and folder list |
| `Ctrl+V` | `paste` | Paste from clipboard into prompt or `[compose]` fields |
| `Ctrl+W` | *(reserved)* | No-op |
| `Ctrl+X` | `cut` | Cut selection in prompt or `[compose]` fields |
| `Ctrl+Y` | `redo` | Redo edit in prompt/`[compose]`, or redo undone Maildir action in `[messages]` |
| `Ctrl+Z` | `undo` | Undo edit in prompt/`[compose]`, or undo last Maildir move/flag action in `[messages]` |
| `Ctrl+\` | `break` | Cancel active prompt, modal, or running background worker (`mbsync`/filter) |

## Context-specific bindings

### `[default]` (global across tabs, sidebar, and panes)

| Key | `TERM=linux` fallback | Widget | Behavior |
| --- | --- | --- | --- |
| `:` | `:` | `command-prompt` | Open the `:` ex-command bar |
| `Tab` / `Shift+Tab` | `Tab` / `Alt+I` | `focus-next` / `focus-prev` | Cycle focus across folder sidebar, message list, and active pane |
| `Alt+1`–`Alt+9`, `Alt+0` | `Alt+1`–`Alt+9`, `Alt+0` | `tab-1`..`tab-9`, `tab-0` | Switch directly to account tab 1–10 |
| `Alt+[` / `Alt+]` | `Alt+[` / `Alt+]` | `tab-prev` / `tab-next` | Switch to previous / next account tab |
| `Escape` | `Escape` | `cancel-mode` | Close overlay/prompt or clear active search highlight |

### `[messages]` (folder sidebar and message list)

| Key | `TERM=linux` fallback | Widget | Behavior |
| --- | --- | --- | --- |
| `Up` / `Down`, `k` / `j` | same | `line-up` / `line-down` | Move selection up / down one message or folder |
| `PageUp` / `PageDown`, `b` / `Space` | same | `page-up` / `page-down` | Scroll message list up / down one page |
| `Home` / `End`, `g` / `G` | same | `buffer-start` / `buffer-end` | Jump to first / last message |
| `/` | `/` | `find` | Search messages in current folder (alias for `Ctrl+F`) |
| `n` / `N` | `n` / `N` | `find-next` / `find-prev` | Jump to next / previous search match |
| `Enter` | `Enter` | `open-message` | Open selected message in `[view]` (or switch to selected folder) |
| `c` | `c` | `compose` | Compose a new message (alias for `Ctrl+N`) |
| `r` / `R` | `r` / `R` | `reply` / `reply-all` | Reply to sender / reply to all recipients |
| `f` | `f` | `forward` | Forward selected message |
| `v` | `v` | `toggle-mark` | Toggle selection mark on current message for bulk actions |
| `u` | `u` | `toggle-read` | Toggle Seen (`S`) flag on selected/marked messages |
| `*` | `*` | `toggle-flag` | Toggle Flagged (`F`) flag on selected/marked messages |
| `d` / `Delete` | `d` / `Delete` | `delete-message` | Move selected/marked messages to Trash (undoable via `Ctrl+Z`) |
| `a` | `a` | `archive-message` | Move selected/marked messages to Archive (undoable via `Ctrl+Z`) |
| `t` | `t` | `toggle-thread` | Collapse or expand the focused message thread |
| `q` | `q` | `quit` | Quit `tam-clif` (alias for `Ctrl+Q`) |

### `[view]` (MIME message pager)

| Key | `TERM=linux` fallback | Widget | Behavior |
| --- | --- | --- | --- |
| `Up` / `Down`, `k` / `j` | same | `line-up` / `line-down` | Scroll message body up / down one line |
| `PageUp` / `PageDown`, `b` / `Space` | same | `page-up` / `page-down` | Scroll message body up / down one page |
| `Home` / `End`, `g` / `G` | same | `buffer-start` / `buffer-end` | Jump to top / bottom of message |
| `/`, `n`, `N` | same | `find`, `find-next`, `find-prev` | Search within message body and step through matches |
| `J` / `K` | `J` / `K` | `next-message` / `prev-message` | Open next / previous message in the folder |
| `Alt+J` / `Alt+K`, `]` / `[` | same | `next-part` / `prev-part` | Cycle to next / previous MIME part |
| `H` | `H` | `toggle-headers` | Toggle full raw RFC 5322 headers |
| `Enter` | `Enter` | `open-part` | Open or pipe the focused MIME attachment |
| `S` | `S` | `save-part` | Save the focused MIME part or attachment (alias for `Ctrl+S`) |
| `r` / `R` / `f` | same | `reply` / `reply-all` / `forward` | Reply / reply-all / forward current message |
| `q` | `q` | `close` | Close `[view]` and return to `[messages]` (alias for `Ctrl+D`) |

### `[compose]` (header editor, attachments, and `$EDITOR` handoff)

| Key | `TERM=linux` fallback | Widget | Behavior |
| --- | --- | --- | --- |
| `Tab` / `Shift+Tab` | `Tab` / `Alt+I` | `field-next` / `field-prev` | Move between header fields (`To`, `Cc`, `Subject`) and attachment list |
| `e`, `Alt+E` | `e`, `Alt+E` | `open-in-editor` | Suspend `tam_tty` and edit body in `$EDITOR` |
| `a` | `a` | `attach-file` | Open attachment picker to attach a file |
| `d` | `d` | `detach-file` | Remove selected attachment |
| `y` | `y` | `send-message` | Submit message via configured `msmtp`/`sendmail` command |
| `p` | `p` | `save-draft` | Postpone message to Drafts (alias for `Ctrl+S`) |
| `q` | `q` | `close` | Discard/close composer with confirmation (alias for `Ctrl+D`) |

### Prompt and `:` ex-command bar (all text-input fields)

Text inputs (`:` ex-command bar, `/` search bar, `[compose]` header
fields) follow the universal editing bindings from
`tam-idol/keys/keybindings.md`:

| Action | Graphical primary | `TERM=linux` / compact fallback | Widget |
| --- | --- | --- | --- |
| Move char | `Left` / `Right` | `Left` / `Right` | `char-left` / `char-right` |
| Move word | `Ctrl+Left` / `Ctrl+Right` | `Alt+,` / `Alt+.` | `word-left` / `word-right` |
| Move line | `Home` / `End` | `Home` / `End` | `line-start` / `line-end` |
| Move buffer | `Ctrl+Home` / `Ctrl+End` | `Alt+<` / `Alt+>` | `buffer-start` / `buffer-end` |
| Delete char | `Backspace` / `Delete` | `Backspace` / `Delete` | `delete-char-left` / `delete-char-right` |
| Delete word | `Ctrl+Backspace` / `Ctrl+Delete` | `Alt+Backspace` / `Alt+D` | `delete-word-left` / `delete-word-right` |
| History / complete | `Up` / `Down`, `Tab` / `Shift+Tab` | `Up` / `Down`, `Tab` / `Alt+I` | `history-prev` / `history-next`, `complete-next` / `complete-prev` |
| Mark selection | `Shift+Left` / `Shift+Right` | `Alt+M` then navigation | `select-char-left` / `select-char-right` / `toggle-mark` |
| Submit / cancel | `Enter` / `Escape` | `Enter` / `Escape` | `accept-line` / `cancel-mode` |

## Planned `binds.ini` format

User overrides live in `$XDG_CONFIG_HOME/tamlinux/tam-clif/binds.ini`
using `tam_ini` sections matching each context. `Meta+` chords are
rejected on load:

```ini
[default]
Alt+P = tab-prev
Alt+N = tab-next

[messages]
F5 = update
q = unbind

[view]
] = next-part
[ = prev-part

[compose]
Alt+E = open-in-editor
```
