---
name: cmux-control
description: Reshape the cmux terminal itself — rename/color/describe the session in the sidebar, group related workspaces, report status pills and progress, arrange panes and splits, notify, peek at or drive sibling sessions, and edit cmux settings. Use whenever the user talks about "this session", "the sidebar", "my tabs/workspaces", "group these", "rename this to X", or otherwise asks to change the shape of their terminal rather than the code in it.
---

# cmux control

cmux is a macOS terminal whose entire UI is drivable from one CLI over a Unix
socket. Anything the user could do with a mouse in the sidebar — rename, color,
group, pin, split, close — you can do with `cmux ...`, from inside the very
session you are running in.

`cmux --help` is the authoritative command list and `cmux <command> --help` the
authoritative flags. This file is a map of *what is worth doing*, not a frozen
copy of the CLI. When something here disagrees with `--help`, `--help` wins.

## 1. Orient before you act

Never pass a ref you did not just read. Refs (`workspace:3`, `surface:8`) are
positional and shift as the user opens and closes things.

```sh
cmux identify --json          # who am I: caller vs. focused window/workspace/pane/surface/tab
cmux tree --all               # the whole instance, one screen
cmux workspace list           # sidebar order, titles, which is selected
cmux workspace-group list     # collapsible groups and their members
cmux sidebar-state            # status pills, progress, log for a workspace
```

`cmux identify` distinguishes **caller** (the surface this shell is in — you)
from **focused** (where the user's eyes are). They are often different: the user
launched you and moved on. Default to acting on the caller unless asked
otherwise, and never steal focus (`--focus false` / `--no-focus`) unless the user
asked to be taken somewhere.

`$CMUX_WORKSPACE_ID`, `$CMUX_SURFACE_ID`, and `$CMUX_TAB_ID` are set in every
cmux terminal and are the default target of most commands, so plain
`cmux workspace-action --action set-color --color Blue` already means "this one".

## 2. The nouns

| Noun | What the user sees | Handle |
|---|---|---|
| window | a macOS window | `window:N` |
| workspace | **one row in the left sidebar** | `workspace:N` |
| workspace group | a collapsible sidebar section, owned by an *anchor* workspace | `workspace_group:N` |
| pane | a split region inside a workspace | `pane:N` |
| surface | one tab inside a pane (terminal, browser, simulator, agent session) | `surface:N` / `tab:N` |

The trap: **"rename this session" is ambiguous.** The sidebar row is the
*workspace*; the tab strip inside it is the *surface*. Renaming one does not
rename the other. When the user means "what I see in the sidebar", they mean the
workspace.

```sh
cmux workspace-action --action rename --title "anvilog · gate"   # sidebar row
cmux rename-tab "build logs"                                     # this tab
cmux rename-window "anvilog"                                     # window title
```

## 3. The semantic contract

Every sidebar channel means exactly one thing, and channels belong to three
axes that must never borrow from each other:

| Axis | Answers | Lifetime | Channels |
|---|---|---|---|
| **Identity** | what is this work? | changes when the work changes | title, group, color, icon, description |
| **State** | what is happening now? | cleared when it ends | pill, progress, spinner, log, notify |
| **Attention** | what needs me? | scarce and temporary | pin, collapse, order, unread badge |

The short version, enough to act on:

- **Title** is `role · object` — the work, not the prompt that started it, under
  ~24 characters, no status words, never repeating the group name.
- **Group** is one repo plus one arc (`anvilog · #72 status`), not a repo bucket.
  Its anchor is a dedicated empty shell, never a working session.
- **Color is the stream and never the status.** One hue per group, set on the
  group *and* on every member — the group color tints only the header, so an
  uncolored member has no stripe at all. Ungrouped rows stay uncolored.
- **Icon** is group-level, naming the kind of arc.
- **Description** is what "done" means for that row. Not a log.
- **Pills, progress, spinner** carry state, and every one of them is a promise
  to clear it. A pill may use color for status precisely because the row's
  color may not.
- **The default is nothing pinned.** Group order already says what matters.
- **Todo lists and unread badges are the user's.** Never write them.

```sh
cmux workspace-action --action rename --title "fix + PR reply"
cmux workspace-action --action set-color --color "#F59E0B"     # the group's hue
cmux workspace-action --action set-description --description "Ship #75 findings. Done = pushed, PR comment posted."
cmux workspace-group set-color workspace_group:1 --hex "#F59E0B"
cmux workspace-group set-icon workspace_group:1 --symbol checkmark.seal
```

Named colors: Red, Crimson, Orange, Amber, Olive, Green, Teal, Aqua, Blue, Navy,
Indigo, Purple, Magenta, Rose, Brown, Charcoal.

`references/sidebar-semantics.md` is the full contract — the per-channel rules,
the failure each one prevents, and the worked examples of titles and groups that
break it. **Read it before reshaping a sidebar**, and state the mapping you used
in your reply so the user can correct it once instead of re-deriving it.

## 4. Grouping: collapse a swarm into a section

Groups are the answer to "I have fourteen agent tabs". Each group is owned by an
anchor workspace, and the group header *is* the anchor's sidebar row.

```sh
cmux workspace-group create --name "anvilog #75" --from <ws>,<ws>,<ws>
cmux workspace-group add --group workspace_group:1 --workspace workspace:5
cmux workspace-group set-color workspace_group:1 --hex "#3B82F6"
cmux workspace-group set-icon  workspace_group:1 --symbol hammer      # SF Symbol
cmux workspace-group collapse workspace_group:1
cmux workspace-group move workspace_group:1 --to-index 0
cmux workspace-group ungroup workspace_group:1                        # dissolve, keep members
```

Good automatic groupings, in rough order of usefulness: by repository (`cwd`),
by ticket or PR number, by lifecycle (in-flight vs. waiting-on-me vs. done).
Read `cmux tree --all` and the surface titles to infer them, then **propose the
grouping before applying it** the first time — grouping visibly rearranges the
user's sidebar.

`cmux workspace-group delete <g> --close-workspaces` closes every member. That
is destructive; only run it on an explicit, unambiguous request.

## 5. Ambient reporting: pills, progress, log, notify

These write into the sidebar without stealing focus — the best way for a
long-running agent to say what it is doing.

```sh
cmux set-status build "compiling" --icon hammer --color "#ff9500" --priority 80
cmux clear-status build
cmux set-progress 0.6 --label "gate: 3/5 checks"
cmux clear-progress
cmux log --level info --source gate "swiftformat clean"
cmux notify --title "Gate failed" --subtitle "anvilog" --body "lint: 2 errors"
cmux workspace loading on   # spinner on the sidebar row
cmux trigger-flash          # attention cue on a surface
```

Use a stable, namespaced key per concern (`gate`, `deploy`) so separate tools do
not clobber each other's pill. Clear what you set when the work ends — a stale
pill is worse than no pill.

## 6. Layout: panes, splits, surfaces

```sh
cmux new-split right --panel pane:1
cmux new-pane --type browser --direction right --url https://github.com/...
cmux new-surface --type terminal --pane pane:2
cmux move-surface --surface surface:7 --pane pane:2 --focus false
cmux split-off --surface surface:7 right
cmux reorder-surface --surface surface:7 --before surface:3
cmux reorder-workspace --workspace workspace:5 --index 0 --dry-run
cmux resize-pane --pane pane:2 -R --amount 10
cmux close-surface --surface surface:7
```

`reorder-workspace` / `reorder-workspaces` accept `--dry-run`. Use it first when
rearranging more than one thing, show the user the plan, then run for real.

## 7. Creating work

```sh
cmux workspace create --name "gate" --cwd ~/projects/anvilog --command "bin/gate"
cmux workspace-group new-workspace workspace_group:1 --placement end
cmux ssh myhost --name "prod logs" --command "journalctl -f"
cmux open ~/projects/anvilog/docs/design.md
cmux markdown open docs/design.md          # formatted viewer, live reload
cmux diff --branch --title "PR #75"        # git diff in a browser split
```

`cmux workspace create --command "..."` is how you hand a long job its own
sidebar row instead of blocking this one.

## 8. Talking to sibling sessions

You can read and drive other surfaces in the same cmux instance. This is
powerful and easy to misuse.

```sh
cmux read-screen --surface surface:4 --scrollback --lines 200   # or capture-pane
cmux send --surface surface:4 "bin/gate\n"
cmux send-key --surface surface:4 Escape
```

Rules: **reading is free, writing is not.** Never `send` into a surface the user
did not point you at — you would be typing into a live agent's or a human's
terminal. Say which surface you are about to write to and why before doing it.

## 9. Settings, sidebars, and config

```sh
cmux settings path && cmux docs settings
cmux config doctor && cmux config validate
cmux reload-config      # reloads cmux.json AND ghostty config, no restart
```

- cmux-owned settings: `~/.config/cmux/cmux.json`.
- Terminal rendering — font, theme, cursor, scrollback, `background-opacity`,
  `background-blur` — belongs in `~/.config/ghostty/config`, **not** cmux.json.
- **Back up `cmux.json` to a timestamped `.bak` before editing it**, always.
- Custom right-sidebar buttons: `.cmux/dock.json` (per repo) or
  `~/.config/cmux/dock.json` — `cmux docs dock`.
- Custom sidebars are runtime-interpreted SwiftUI in `~/.config/cmux/sidebars/`
  — `cmux docs sidebars`, then `cmux sidebar validate|reload|select`.

## 10. Reacting to the instance

`cmux events` streams newline-delimited JSON for everything happening in cmux —
notifications, workspace changes, feed items. It is the hook for "watch until X,
then reshape".

```sh
cmux events --category notification --limit 1
cmux events --cursor-file ~/.cache/cmux/events.seq --reconnect
```

Run long streams in the background, never in the foreground of a turn.

## Rules

1. Read state (`identify`, `tree`, `workspace list`) before every mutation. Refs go stale.
2. Do not move the user's focus or reorder their sidebar as a side effect of some
   other task. Reshaping is the task or it does not happen.
3. Destructive verbs — `close-workspace`, `close-others`, `close-surface`,
   `workspace-group delete --close-workspaces`, `browser history clear` — need an
   explicit request naming the target. Never as cleanup.
4. **The workspace todo list belongs to the user.** `cmux todo` is their
   checklist, shown in their sidebar. Only touch it when asked in so many words;
   track your own plans with your own todo tool.
5. `cmux send` into another surface is typing on someone else's keyboard.
   Announce it first.
6. Back up `cmux.json` before editing; `cmux reload-config` after.
7. Prefer `--dry-run` and `--focus false` / `--no-focus` wherever they exist.
8. When unsure whether "this session" means the workspace or the tab, do the
   workspace — that is the sidebar — and say which you did.

`references/cheatsheet.md` has the fuller command inventory captured from
cmux 0.64.22.
