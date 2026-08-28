---
description: Clean up the cmux sidebar so it matches its semantic contract — apply the mechanical fixes, then hand what needs judgment to a cheap model that reads each session and retitles it. Use when the sidebar has drifted, when a Stop hook reports items needing judgment, or when the user asks to tidy, clean up, or re-organize their sidebar.
---

# Tidy the sidebar

Two passes with a clean split: **what is decidable from state, and what needs
someone to look.**

```sh
cmux-sidebar-sync --check    # what is wrong, changing nothing
cmux-sidebar-sync            # fix the mechanical half, report the rest
```

`bin/cmux-sidebar-sync` enforces the rules that follow from state alone — color
follows group, ungrouped rows carry no hue, anchor titles match group names —
and reports what it cannot decide: a title that reads like an opening prompt, a
row holding two streams, a group with no members. It costs no tokens and is safe
to run on every turn; the plugin's `Stop` hook already does, in `--hook` mode,
where it applies fixes silently and prints one line if judgment is needed.

Everything it reports needs a look at what the sessions are actually doing. That
is the second pass, and it is the only part that costs anything.

## Run it

1. **`cmux-sidebar-sync`** — apply the mechanical fixes and read the drift list.
   Exit 0 means done; there is nothing further to do and you should say so
   rather than reorganizing something that is already correct.

2. **Dispatch the groundskeeper** for the drift, using the `sidebar-groundskeeper`
   agent — it runs on a cheap model because the work is "read four screens,
   write four short titles", not reasoning. Hand it the drift list verbatim and
   the workspace refs. It reads each session with `cmux read-screen`, renames to
   `role · object`, fills in group color and icon, and reports what it left.

3. **Relay what changed** — the renames, and anything it declined to decide.
   Group splits, closes, and reorders are not its job and come back as reports;
   act on them yourself only if the user asks.

## When to run the expensive pass

On demand, and when the Stop hook says judgment is needed — not automatically.
The cheap pass is idempotent and invisible, so running it constantly is free.
The judgment pass **renames the user's rows**, and a rename that happens without
them watching is indistinguishable from the sidebar lying to them: they look
away with `What's next` and look back at something else. Let the hook surface
that cleanup is available and let a person say go.

The one exception worth offering: after a burst of workspace creation — a swarm
of agents just spawned — the sidebar is auto-named and uninformative, and
tidying immediately is what makes it readable at all.

## Rules

1. The deterministic pass runs first, always. Never spend a model on something
   `cmux-sidebar-sync` already fixes.
2. Exit 0 means stop. Do not improve a compliant sidebar.
3. The groundskeeper owns titles, group membership, group color and icon.
   Closing, splitting, reordering, and pinning stay with you and the user.
4. Report renames back to the user explicitly. They have to be able to find
   their rows again.
