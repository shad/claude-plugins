# cmux-control

A skill for driving the cmux terminal from inside a Claude
Code session running in it.

cmux exposes its entire UI over a CLI, so anything you could do with the mouse
in the sidebar — rename, color, group, pin, split, close — Claude can do for
you. This skill teaches it *what is worth doing*: naming conventions that make
a sidebar of a dozen parallel agents readable, grouping a swarm of workspaces
into a collapsible section, and reporting progress with status pills instead of
stealing focus.

It also encodes the guardrails: read state before mutating (refs go stale),
never move the user's focus as a side effect, and treat `cmux send` into another
surface as typing on someone else's keyboard.

## Install

```
/plugin marketplace add shad/claude-plugins
/plugin install cmux-control@shad
```

## What's in it

| Component | Does |
|---|---|
| `cmux-control` skill | the contract, and how to reshape a sidebar under it |
| `references/sidebar-semantics.md` | the full per-channel contract |
| `bin/cmux-sidebar-sync` | deterministic pass — fixes what state decides, reports the rest |
| `tidy-sidebar` skill | runs both passes in order |
| `sidebar-groundskeeper` agent | the cheap-model pass: reads sessions, retitles rows |
| `Stop` hook | runs the deterministic pass after every turn, silently |

## The sidebar contract

Every channel means one thing, and channels never borrow across axes:

- **Identity** — title, group, color, icon, description. Changes when the work
  changes. Color is the *stream*, never the status.
- **State** — pills, progress, spinner, log, notify. Every one is a promise to
  clear it.
- **Attention** — pin, collapse, order, unread badge. Scarce by definition; the
  default is nothing pinned.

`references/sidebar-semantics.md` has the per-channel rules and the failure each
one prevents.

## Use

The skill is model-invoked — just talk about the terminal:

- "rename this session to PR #75 review"
- "group these four workspaces by repo"
- "put a status pill up while the build runs"

## Requires

cmux, and a session running inside it.
