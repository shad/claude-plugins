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

## Use

The skill is model-invoked — just talk about the terminal:

- "rename this session to PR #75 review"
- "group these four workspaces by repo"
- "put a status pill up while the build runs"

## Requires

cmux, and a session running inside it.
