---
description: Clean up the cmux sidebar — group rows by the feature their session is working on, refresh PR pills, and fix titles that name the opening ask instead of the work. Use when the sidebar has drifted, when the Stop hook reports changes, or when the user asks to tidy or re-organize their sidebar.
---

# Tidy the sidebar

Two passes. One is free and runs on its own. The other costs a model and runs
when asked.

## The free pass

```sh
cmux-sidebar-sync --check    # report only
cmux-sidebar-sync            # group rows, set PR pills
```

It reads each row's transcript, finds the last worktree the session entered, and
puts the row in that feature's group. It creates groups. It never moves a row
out or undoes a placement. PR pills come from one cached `gh` call per repo.

The plugin's Stop hook already runs this after every turn. It costs nothing, so
do not spend a model on anything it handles.

## The paid pass

cmux names every row itself, every turn, free. About a third of those names are
the opening ask rather than the work — `Post findings on PR` for a session that
posted them an hour ago.

Fix those with the `sidebar-groundskeeper` agent. It runs on a cheap model
because the work is "read four screens, write four short titles." Hand it the
rows to look at. It reads each session and renames to `role · object`.

Run it when the user asks. Not automatically. A rename the user did not watch
happen is a sidebar that lies to them — they look away with `What's next` and
look back at something else.

The exception: right after a swarm spawns, every row is auto-named and the
sidebar is unreadable. Tidying then is what makes it useful at all.

## Rules

1. Free pass first. Never spend a model on what it already does.
2. Nothing to fix means say so. Do not improve a sidebar that is fine.
3. The groundskeeper owns titles. Grouping, closing, and splitting stay with you.
4. Report every rename. The user has to find their work again.
