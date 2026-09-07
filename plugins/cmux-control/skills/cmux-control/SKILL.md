---
name: cmux-control
description: Reshape the cmux terminal itself — rename/color/describe the session in the sidebar, group related workspaces, report status pills and progress, arrange panes and splits, notify, peek at or drive sibling sessions, and edit cmux settings. Use whenever the user talks about "this session", "the sidebar", "my tabs/workspaces", "group these", "rename this to X", or otherwise asks to change the shape of their terminal rather than the code in it.
---

# cmux control

cmux is a macOS terminal whose UI is drivable from one CLI. Anything the user
could do with a mouse — rename, group, split, close — you can do from inside the
session you are running in.

`cmux --help` is the authoritative command list. This file says what is worth
doing. When they disagree, `--help` wins.

## 1. Orient first

Refs are positional. They shift when the user opens and closes things. Never
pass a ref you did not just read.

```sh
cmux identify --json          # caller vs. focused
cmux tree --all               # the whole instance
cmux workspace list --json    # titles, groups, cwd, pinned
cmux workspace-group list --json
```

`caller` is the surface this shell is in. `focused` is where the user's eyes
are. They are often different. Act on the caller. Never steal focus.

`$CMUX_WORKSPACE_ID` and `$CMUX_SURFACE_ID` are set in every cmux terminal, so
plain `cmux workspace-action --action rename --title x` means "this one".

## 2. The nouns

| Noun | The user sees | Handle |
|---|---|---|
| window | a macOS window | `window:N` |
| workspace | **one row in the sidebar** | `workspace:N` |
| workspace group | a collapsible section | `workspace_group:N` |
| pane | a split region | `pane:N` |
| surface | one tab in a pane | `surface:N` / `tab:N` |

"Rename this session" is ambiguous. The sidebar row is the workspace. The tab
strip inside it is the surface. Renaming one does not rename the other. Default
to the workspace and say which you did.

```sh
cmux workspace-action --action rename --title "gate"   # sidebar row
cmux rename-tab "build logs"                           # this tab
```

## 3. The contract

Read `references/sidebar-semantics.md` before reshaping anything. The short
version:

- **A group is one worktree.** One feature. Rows with no worktree evidence go to
  that repo's `repo · main` bucket.
- **Grouping is additive.** Create and add. Never move a row out, dissolve a
  group, or re-parent what a person placed.
- **Group name is `repo · feature`** — the directory basename. No PR number; the
  PR is state.
- **Titles come from cmux.** It names every row every turn, free. Fix one only
  when asked.
- **Pills carry state** and every one is a promise to clear it.
- **Notify only when blocked or done.** This is the only lever on order.
- **Never reorder. Never color. Nothing pinned.** cmux owns position. Color is
  retired. Pins are for a real moment, not decoration.
- **Todo lists and unread badges are the user's.**

## 4. Finding a row's feature

Shell cwd will not tell you. Every session reads in the main checkout and writes
elsewhere, so cwd says `anvilog` for all of them.

A `PostToolUse` hook records what each session touches — any Edit, Write, or
Bash naming a worktree — under `~/.cache/cmux-sidebar-sync/session/<id>`. That
is ground truth and needs no convention. The transcript is the fallback:

```sh
grep -o '"cwd":"[^"]*worktrees/[^"]*"' ~/.claude/projects/<slug>/<session>.jsonl | tail -1
```

It only sees `EnterWorktree`. A session using `git -C` records nothing there.

```sh
cmux workspace-group create --name "anvilog · window-planner" --cwd <path>
cmux workspace-group add --group workspace_group:1 --workspace workspace:5
cmux workspace-group set-icon workspace_group:1 --symbol hammer
cmux workspace-group collapse workspace_group:1
```

`create` without `--from` makes an empty anchor shell. Use it. An anchor that is
a real session hides that session's row.

`workspace-group delete <g> --close-workspaces` closes every member. Only on an
explicit request naming the target.

## 5. Reporting without interrupting

```sh
cmux set-status pr "#71 draft" --icon arrow.triangle.branch
cmux clear-status pr
cmux set-progress 0.6 --label "gate: 3/5"
cmux workspace loading on
cmux log --level info --source gate "swiftformat clean"
cmux notify --title "Gate failed" --body "lint: 2 errors"
```

Namespace each key. Clear what you set. `log` is free; `notify` costs the user a
context switch, so it fires only when you are blocked on them or done.

## 6. Layout

```sh
cmux new-split right --panel pane:1
cmux new-pane --type browser --direction right --url https://github.com/...
cmux move-tab-to-new-workspace --tab tab:2 --workspace workspace:2 --title "impl"
cmux move-surface --surface surface:7 --pane pane:2 --focus false
cmux close-surface --surface surface:7
```

One row is one job. Split a tab out when a row has grown a second stream — the
tell is that its title describes only one of its tabs.

## 7. Creating work

```sh
cmux workspace create --name "gate" --cwd ~/projects/anvilog --command "bin/gate"
cmux ssh myhost --name "prod logs" --command "journalctl -f"
cmux markdown open docs/design.md
cmux diff --branch --title "PR #75"
```

`workspace create --command` hands a long job its own row instead of blocking
this one.

## 8. Sibling sessions

```sh
cmux read-screen --surface surface:4 --scrollback --lines 200
cmux send --surface surface:4 "bin/gate\n"
```

Reading is free. Writing is not. `send` is typing on someone else's keyboard.
Never into a surface the user did not point you at, and say so first.

## 9. Settings

```sh
cmux settings path && cmux docs settings
cmux reload-config      # cmux.json and ghostty, no restart
```

cmux.json is `~/.config/cmux/cmux.json`. Back it up to a timestamped `.bak`
before editing. Font, theme, opacity, and blur live in
`~/.config/ghostty/config`, not cmux.json.

`app.reorderOnNotification` is what floats a notifying row to the top. Leave it
on. It is the ordering.

## 10. Watching

```sh
cmux events --category notification --limit 1
cmux events --cursor-file ~/.cache/cmux/events.seq --reconnect
```

Run long streams in the background, never in the foreground of a turn.

## Rules

1. Read state before every mutation. Refs go stale.
2. Reshaping is the task or it does not happen. Never as a side effect.
3. Grouping is additive. Never undo a person's placement.
4. Destructive verbs need an explicit request naming the target. Never as cleanup.
5. The todo list is the user's. Track your own plans with your own tools.
6. `send` into another surface is typing on someone else's keyboard. Announce it.
7. Say what you changed. The user has to find their work again.

`references/sidebar-semantics.md` is the full contract.
`references/cheatsheet.md` is the command inventory from cmux 0.64.22.
