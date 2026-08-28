---
description: Package an existing skill, agent, or hook into a plugin in a local plugin marketplace repo — creating the plugin directory and manifest, adding the marketplace.json entry, validating, documenting, committing, and installing it. Use when the user wants to publish, share, or "turn into a plugin" something that currently lives in ~/.claude/, or wants to add a new plugin to their marketplace.
---

# Package a plugin into a marketplace

Takes something that already works locally — a skill in `~/.claude/skills/`, an
agent, a hook — and makes it installable by other people, in one pass, ending
with it committed, pushed, and installed for the user.

The failure mode this skill exists to prevent is a half-published plugin: a
directory in the repo with no marketplace entry, or a marketplace entry pointing
at a path that doesn't exist, or the original still sitting in `~/.claude/skills/`
loading a second copy of the same skill. Finish every step or back all of them out.

## 1. Find the marketplace

Local marketplaces are registered as `"source": "directory"`. Read them:

```sh
cat ~/.claude/plugins/known_marketplaces.json
```

Take the `path` of the directory-sourced entry. If there are several, ask which.
If there are none, the user has no local marketplace repo — say so and offer to
create one (§8) rather than guessing at a path.

Confirm the repo is clean before you write anything (`git status --short`).
Uncommitted work belonging to the user should be left alone; a dirty tree also
blocks `claude plugin tag` later.

## 2. Decide the name and the shape

The plugin name is the install identity, the namespace for its skills
(`/name:skill`), and the directory under `plugins/`. Kebab-case, no `claude-` or
`-plugin` padding.

| The user has | Layout |
|---|---|
| one skill | `plugins/<name>/skills/<name>/SKILL.md` |
| several related skills | `plugins/<name>/skills/<skill>/SKILL.md` each |
| an agent | `plugins/<name>/agents/<agent>.md` |
| hooks | `plugins/<name>/hooks/hooks.json` |

Name the plugin after the *capability*, not the first skill in it, when you can
tell it will grow. `/cmux-control:cmux-control` is the price of naming a plugin
after its only skill; that is acceptable, just decide it deliberately.

**Never put `skills/`, `agents/`, `hooks/`, or `commands/` inside
`.claude-plugin/`.** Only `plugin.json` goes there. This is the single most
common structural mistake.

## 3. Copy in the source material

```sh
mkdir -p plugins/<name>/.claude-plugin plugins/<name>/skills/<skill>
cp -R ~/.claude/skills/<skill>/. plugins/<name>/skills/<skill>/
```

Copy — do not move yet. §7 removes the original only after everything validates.

Then read the `SKILL.md` you just copied and check two things:

- **The `description` frontmatter is the whole discovery mechanism.** It is what
  Claude reads to decide whether to fire the skill. It should name the concrete
  triggers — the words a user would actually say — not summarize the skill's
  contents. Rewrite it if it is vague.
- **Nothing machine-specific leaks.** Absolute paths under `/Users/<someone>`,
  hostnames, tokens, internal URLs, coworkers' names. Grep for them. This is
  about to be public.

Drop `name:` from the frontmatter if present — the directory name and the plugin
namespace supply it.

## 4. Write the plugin manifest

`plugins/<name>/.claude-plugin/plugin.json`:

```json
{
  "name": "<name>",
  "description": "<one sentence, shown in the plugin manager>",
  "version": "0.1.0",
  "author": { "name": "<from an existing plugin in the repo>" },
  "homepage": "<the docs page, if the repo has one>",
  "repository": "<the repo URL>",
  "license": "<match the repo's LICENSE>",
  "keywords": ["..."]
}
```

Start at `0.1.0`. Copy `author`, `homepage`, `license` from a sibling plugin so
the repo stays internally consistent.

## 5. Add the marketplace entry

Append to `plugins` in `.claude-plugin/marketplace.json`:

```json
{
  "name": "<name>",
  "source": "./plugins/<name>",
  "description": "<same sentence as the plugin manifest>",
  "version": "0.1.0",
  "author": { "name": "..." },
  "homepage": "...",
  "license": "...",
  "keywords": ["..."],
  "category": "productivity"
}
```

The relative `source` resolves against the marketplace root, which is why this
repo layout works at all. **`version` here must equal `version` in
`plugin.json`** — `claude plugin tag` refuses to tag when they disagree, and
users get stale installs when they drift.

Edit the JSON with a small script rather than by hand-splicing text, and keep the
existing 2-space indent and key order.

## 6. Validate before you commit

```sh
claude plugin validate .                    # the marketplace
claude plugin validate ./plugins/<name>     # the plugin
claude plugin validate ./plugins/<name>/skills   # skills, agents, commands
```

All three must pass. Warnings are worth fixing — they are usually a missing
`description` that would have made the plugin harder to find.

Then load it for real in a throwaway session before believing it works:

```sh
claude --plugin-dir ./plugins/<name>
```

`--plugin-dir` beats an installed copy of the same name for that session, so
this is also how you test a change to a plugin you already have installed.

## 7. Retire the original

An installed plugin skill and a personal `~/.claude/skills/` skill of the same
name **both load** — plugin skills are namespaced, so they do not override, they
duplicate. Two copies of the same instructions, twice the always-on tokens.

So once §6 passes, move the original aside:

```sh
mv ~/.claude/skills/<skill> /tmp/<skill>.pre-plugin-backup
```

`diff -r` the two first and say so if they differ — the local copy may have
edits that never made it into your copy. Move rather than delete, and tell the
user where the backup went.

## 8. Document, commit, install

**Docs.** If the repo's README has a plugin table, add a row. If the user has a
docs page for the marketplace — for this repo that is `plugins.md` in the
`shadr.us` site repo — add a section in the same shape as the existing ones:
heading linking to the plugin directory, the install command in a fenced block,
two short paragraphs on what it does and why, and a `*Requires*:` line if it
depends on anything external. Match the surrounding voice; do not introduce
headings, badges, or tables the page doesn't already use.

**Commit** the marketplace repo, then the site repo separately. Push both.

**Install** for the user:

```sh
claude plugin install <name>@<marketplace>
claude plugin details <name>@<marketplace>    # confirm the components loaded
```

`details` prints the component inventory and the always-on token cost. Read it
back to the user — if it says `Skills (0)`, the layout is wrong and §2's warning
is why.

Note that install *copies* into `~/.claude/plugins/cache/<marketplace>/<name>/<version>/`.
Editing the repo afterwards does not change the installed copy; that is what
`--plugin-dir` and `release-plugin` are for.

## Creating a marketplace from scratch

If §1 found no local marketplace, a new one is four files:

```
.claude-plugin/marketplace.json    { name, description, owner{name,url}, plugins: [] }
README.md                          install instructions and a plugin table
LICENSE
plugins/
```

Then `claude plugin marketplace add <path>` to register the local copy, and
`gh repo create <name> --public --source=. --push` to publish it. Others add it
with `/plugin marketplace add <owner>/<repo>`.

## Rules

1. Validate before committing, install after pushing. Never the other way round.
2. `version` in `plugin.json` and in the marketplace entry are one number in two
   files. Change both or neither.
3. Read the `SKILL.md` you are about to publish, all of it, and grep it for
   machine-specific paths and names. Publishing is not reversible.
4. Move the original out of `~/.claude/skills/`, never delete it, and only after
   validation passes.
5. Do not commit the user's unrelated working-tree changes along with yours.
6. If a step fails, undo the earlier steps rather than leaving a plugin that is
   half in the marketplace.
