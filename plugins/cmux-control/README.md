# cmux-control

Drive the cmux terminal from a Claude session running inside it.

cmux exposes its whole UI over a CLI, so a session can reshape the terminal it
lives in. This plugin decides what is worth doing with that, and what each part
of the sidebar means.

## Install

```
/plugin marketplace add shad/claude-plugins
/plugin install cmux-control@shad
```

## The contract

Every channel means one thing:

- **A group is one worktree.** One feature. Rows in the main checkout go to a
  `repo · main` bucket.
- **Grouping is additive.** Create and add, never move out or re-parent.
- **Titles come from cmux**, free, every turn. Fix one only when asked.
- **Pills carry PR state**, from one cached `gh` call per repo.
- **Notify only when blocked or done.** It is the only lever on order.
- **Never reorder. Never color. Nothing pinned.**

Shell cwd cannot tell you a row's feature — every session reads in the main
checkout and writes elsewhere. So a hook records what each session touches. That
is ground truth, and it needs no convention: a session reaches a worktree by
`EnterWorktree`, `git -C`, or absolute path, and all three touch files.

`skills/cmux-control/references/sidebar-semantics.md` is the full contract.

## What's in it

| Component | Does |
|---|---|
| `cmux-control` skill | the contract, and how to reshape a sidebar under it |
| `bin/cmux-worktree-note` | records which worktree a session touches |
| `bin/cmux-sidebar-sync` | groups rows by feature, sets PR pills |
| hooks | `PostToolUse` records worktrees, `Stop` runs the sync |
| `tidy-sidebar` skill | runs both passes in order |
| `sidebar-groundskeeper` agent | cheap-model pass: reads sessions, fixes titles |

## Use

Talk about the terminal:

- "group these by feature"
- "put a pill up while the build runs"
- "tidy the sidebar"

## Requires

cmux. `gh` for PR pills.
