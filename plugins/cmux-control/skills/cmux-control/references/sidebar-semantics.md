# Sidebar semantics

The sidebar shows a dozen parallel agents at a glance. Each channel means one
thing. This file says what.

Most of it is derived, not authored. Nobody names a group. Nobody picks a color.
The work is in what you *don't* touch.

## The three axes

| Axis | Answers | Lifetime | Channels |
|---|---|---|---|
| **Identity** | what is this work? | changes when the work changes | group, group name, icon, title, description |
| **State** | what is happening? | cleared when it ends | pill, progress, spinner, notify |
| **Attention** | what needs me? | cmux decides | position |

A channel never expresses another axis. A title never says `WIP`. A pill never
says what a row *is*.

---

## Identity

### Group — one worktree

A group is one feature: one directory under `.worktrees/` (or
`.claude/worktrees/`).

Every repo also gets a `repo · main` bucket. Rows with no worktree evidence go
there — they are working in the main checkout, and that is a place, so it is a
group. A bucket carries no PR pill; there is no branch to join on.

A row's shell cwd will not tell you its feature. Every session reads in the main
checkout and writes elsewhere, so cwd says `anvilog` for all of them.

Two sources, in order.

**What the session touched.** A `PostToolUse` hook watches every Edit, Write, and
Bash call. When a path or command names a worktree, it records the feature under
`~/.cache/cmux-sidebar-sync/session/<session-id>`. This is ground truth. A
session reaches a worktree by `EnterWorktree`, by `git -C`, or by absolute path,
and no convention makes those the same — but all three touch files.

**The transcript**, as a fallback:

```sh
grep -o '"cwd":"[^"]*worktrees/[^"]*"' ~/.claude/projects/<slug>/<session>.jsonl | tail -1
```

The last worktree the session entered. Scan for the last *worktree* cwd, not the
last cwd — a session that finished has returned to main. 13ms.

The transcript only knows about `EnterWorktree`. A session that works through
`git -C` records nothing, which is why the hook exists. Instructions do not fix
this. A repo can say "never work in the main checkout" in bold, three times, and
a session will still edit through an absolute path and look like main.

**Grouping is additive.** Create a group, add a row. Never move a row out. Never
dissolve a group. Never re-parent something a person placed. A wrong guess
becomes clutter you can drag away, and it stays dragged.

### Group name — `repo · feature`

The directory basename. `anvilog · window-planner`. It never changes, because
the directory doesn't.

No PR number in the name. The PR is state; it belongs on a pill.

### Group anchor — an empty shell

The header is the anchor's row. Use a dedicated empty shell in the feature's
directory. Make a real session the anchor and its row disappears.

Title the anchor the same as the group.

### Title — what the session is doing

cmux writes these. A Stop hook names every row every turn, free.

It names the opening ask about a third of the time — `Post findings on PR` for a
session that posted them an hour ago and is now re-reviewing a fix. Live with
it. Fix a row when the user asks, not on your own initiative.

When you do fix one: `role · object`, under 24 characters, no status words, and
never repeat the group name.

### Icon — the kind of work

Group level. Set once.

| Symbol | Work |
|---|---|
| `checkmark.seal` | review |
| `arrow.triangle.branch` | a branch under construction |
| `globe` | a site or deploy |
| `hammer` | build or tooling |
| `magnifyingglass` | investigation, no PR yet |

Never on a row. Never for state.

### Description — what "done" means

One or two lines. A contract, not a log.

```
Ship the #75 findings. Done = pushed, comment posted, card moved.
```

Rewrite it when the finish line moves. Never append to it.

---

## State

Every state channel is a promise to clear it. A stale pill is worse than no
pill. It reports something untrue, and the user cannot tell without opening the
row — which is the cost the sidebar exists to avoid.

### Pill — the PR

`cmux set-status pr "#71 draft"`. Number and state, from `gh`.

Pills go on rows, not on group headers. A header draws its name, its icon, and
nothing else — a pill set on the anchor is stored and never seen. So every
member of a feature carries its PR pill.

One call per repo, not per row:

```sh
gh pr list --state all --limit 50 \
  --json number,title,headRefName,state,isDraft,reviewDecision,labels
```

0.55s, covers every worktree. Cache it 60 seconds, keyed on the repo root. Map a
row to its PR through its branch: `cat .git/worktrees/<slug>/HEAD` is 2ms and
needs no network. No PR number is stored locally, so the branch is the only
join.

Other pills follow the same rule: a namespaced key, a fact worth knowing without
opening the row, cleared the moment it stops being true.

### Progress and spinner

`set-progress` needs a real denominator. `workspace loading on` is for waits
without one. Both end when the work does.

### Notify — and why it is also the order

`cmux notify` when the work is **blocked on the user** or **finished**. Never for
progress.

This is the only lever on sidebar order. cmux floats a notifying row toward the
top and slides its group up with it. Notify honestly and the sidebar sorts
itself. Notify for progress and it sorts by noise.

---

## What we do not touch

**Position.** cmux owns it. Never call `reorder-workspace`.

**Color.** Unused. It was the stream, and the stream is now the group. Leave rows
uncolored until color earns a job worth having.

**Pins.** None. Group order already says what matters. A pin means "this one,
above the rest," and only while few things carry it. Reserve it for a real
moment, and unpin when the moment passes.

**Todo lists and unread badges.** The user's. A badge is their record of what
they still owe a look.

---

## When you reshape a sidebar

Say what you did. Which rows moved, what you renamed, what you left alone. The
user has to find their work again.
