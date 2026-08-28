# Sidebar semantics

The sidebar is a display with a fixed set of channels — title, color, icon,
group, order, pill, progress, spinner, badge, pin — and only as much meaning as
you agree to give them. This file is that agreement. Follow it exactly; an
element used off-contract is worse than an unused one, because the user has to
stop and decide what it means this time.

## The three axes

Every channel belongs to exactly one axis, and **no channel may express a
different axis than its own.**

| Axis | Answers | Lifetime | Channels |
|---|---|---|---|
| **Identity** | what is this work? | changes only when the work changes | title, group, group name, color, icon, description |
| **State** | what is happening right now? | must be cleared when it ends | status pill, progress, spinner, log, notify, flash |
| **Attention** | what needs me? | scarce, temporary, earned | pin, collapse, order, unread badge |

The failure this prevents: color drifting into meaning "failing", a title
growing a `[WIP]`, a status pill left up for two days until it reads as part of
the row's name. Each of those borrows a channel from another axis and quietly
destroys the one it borrowed from.

---

## Identity

### Workspace row — one unit of work

One row is one piece of work with one owner: the Claude session in **tab
position 1**. Other tabs in the row are companions to that work — a shell, a log
tail, a browser on the PR. A second Claude session working a different problem is
a second row, not a second tab.

Split with `cmux move-tab-to-new-workspace --tab <tab> --workspace <ws>` when a
row has grown a second stream. The tell is that the row's title only describes
one of its tabs.

### Title — `role · object`

What the session *is doing*, not the prompt that started it.

| Bad | Why | Good |
|---|---|---|
| `What's next` | the opening prompt, not the work | `iPad test plan` |
| `Settings review on 75` | what it was asked, not what it became | `fix + PR reply` |
| `anvilog PR #75 status fixes [WIP]` | repeats the group, carries state | `fix + PR reply` |

Rules: under ~24 characters, since the sidebar truncates. No status words — no
`WIP`, `done`, `blocked`, no emoji standing in for state. Never repeat the group
name; the row is already inside it. Retitle when the work turns into something
else, and only then.

### Group — one stream

A group is **one repo plus one arc**: a ticket, a PR, a release. Named
`repo · #N topic` (`anvilog · #72 status`).

A group is *not* a repo bucket. Two unrelated efforts in one repo are two groups
— that is the whole point, and collapsing them back into `anvilog` is the most
tempting way to lose the distinction. A group is also not a "misc" tray: work
that belongs to no stream stays ungrouped, where its being loose is accurate.

### Group anchor — a dedicated empty shell

The header *is* the anchor's row, so the anchor should be an empty shell in the
stream's directory, never a working session. Making a real session the anchor
hides that session's row and its state.

Give the anchor the same title as the group name, so it reads correctly if the
group is ever dissolved.

### Color — the stream, and nothing else

**One hue per group.** The group carries it (`workspace-group set-color --hex`)
and every member carries the same hue (`workspace-action --action set-color`) —
the group color tints only the header, so members without their own color lose
their stripe entirely.

Color is identity. It never means passing, failing, running, or urgent. Status
has its own channel and it is the pill. A row's hue answers "which stream is
this?" from across the screen, at a glance, without reading — that is the only
job it has, and it can only do it while the mapping holds still.

Ungrouped rows get no color. Being uncolored is what "belongs to no stream"
looks like.

### Icon — the kind of stream

Group-level only, an SF Symbol naming what kind of arc this is:

| Symbol | Stream |
|---|---|
| `checkmark.seal` | review / fixing findings |
| `arrow.triangle.branch` | a branch or PR under construction |
| `globe` | a site or deploy target |
| `hammer` | a build or tooling effort |
| `magnifyingglass` | an investigation with no PR yet |

Never put an icon on a member row and never use one for state. A row that grows
a 🔥 has stolen the pill's job.

### Description — the standing intent

`workspace-action --action set-description` holds **what "done" means for this
row**, in a line or two. It is a contract, not a log:

```
Ship the #75 review findings. Done = fixes pushed, PR comment posted,
card moved to Ready to merge.
```

Never append progress to it. Never let it become a transcript. If it no longer
describes the finish line, rewrite it.

---

## State

Everything here is a promise to clean up. A stale state channel is worse than an
empty one: it reports something that is no longer true, and the user cannot tell
without opening the row — which is the exact cost the sidebar exists to avoid.

### Status pill — a fact that outlives one command

`cmux set-status <key> <value>` for a fact about the work that persists across
commands and is worth knowing without opening the row: `gate=passing`,
`deploy=v1.2.3`, `pr=#75 needs-you`.

Namespace the key per concern so tools don't clobber each other, and **clear it
the moment it stops being true** (`cmux clear-status <key>`). A pill is where
color *may* encode state — green passing, amber running, red failing — because a
pill is short-lived by contract. That is exactly why the row's own color must
not.

Not for: what the session is doing right now (that's the row's own live status),
or anything that fits in the title.

### Progress — a bounded, countable run

`cmux set-progress <0..1> --label "gate: 3/5 checks"` only when there is a real
denominator. No fake fractions for unbounded work; that is the spinner's job.
Clear it when the run ends, pass or fail.

### Spinner — an unbounded wait

`cmux workspace loading on` for "working, no idea how long". Off the moment it
resolves.

### Log and notify — timeline vs. interruption

`cmux log` writes a timeline for reading afterwards; it costs the user nothing.
`cmux notify` crosses into their attention and costs them a context switch —
so it fires only when the work is **blocked on them** or **finished**. Progress
is not a notification. `trigger-flash` is the weaker in-view version of the same
decision.

---

## Attention

### Pin — scarce by definition

A pin holds a row or group at the top of the sidebar while everything around it
churns. It has no meaning beyond scarcity: it says "this one, above the rest",
and it says it only while few things carry it.

**The default is nothing pinned.** Group order already encodes what matters;
spending a pin on top of it buys nothing and spends the signal. Legitimate uses
are temporary states with an end:

- a parking spot you return to while opening and closing investigative tabs
- a stake in the ground for the duration of one long piece of work

Both are unpinned when the moment passes. A pin that has been up for a week has
stopped saying anything. Never pin as decoration, and never pin more than one
thing without the user asking for exactly that.

### Collapse — a stream that is not today's

Collapse a group that is real but idle: reference, not noise. Do not collapse
something merely because the sidebar is long — that hides work rather than
organizing it.

### Order — where attention is, not what is important

Groups run in order of the user's current attention: what they are working in
now, first. Rows within a group run in the order the work flows
(`impl → fix → re-review`), not by status.

Reordering more than one thing gets `--dry-run` and a look first.

### Unread badge — cmux's, not yours

The count is the terminal's own record of output the user hasn't seen.
`mark-read` / `mark-unread` are the user's controls. Do not clear a badge to
tidy the sidebar; you would be erasing their record of what they still owe a
look.

---

## The user's channels

Two things in the sidebar are never yours to write:

- **The todo list.** `cmux todo` is the user's checklist. Track your own plans
  with your own tools.
- **The unread badges**, per above.

---

## Applying the scheme

When you reshape a sidebar, state the mapping you used in your reply — the hues,
what each group means, what you named things — so the user can correct it once
rather than re-derive it every time they look. And when you touch one row, check
that the scheme still holds for the others; a convention that applies to half
the sidebar is not a convention.
