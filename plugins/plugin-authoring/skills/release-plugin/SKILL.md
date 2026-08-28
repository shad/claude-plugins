---
description: Cut a release of a plugin in a local marketplace repo — bump the version in both manifests, validate, tag, push, and update the installed copy. Use when the user has edited a plugin in their marketplace and wants to ship the change, release a new version, or get their installed copy up to date.
---

# Release a plugin

Edits to a plugin in the marketplace repo do **not** reach the user's installed
copy. Install pins a version and copies the tree to
`~/.claude/plugins/cache/<marketplace>/<name>/<version>/`. Shipping a change means
bumping the version, pushing, and updating — this skill is that loop.

If the user is still iterating and just wants to see their edit work, they do not
need a release. Point them at `claude --plugin-dir ./plugins/<name>` plus
`/reload-plugins`, which loads the working tree directly and takes precedence
over the installed copy.

## 1. Establish what changed

```sh
cd <marketplace root>            # from ~/.claude/plugins/known_marketplaces.json
git status --short
git diff --stat
git log --oneline -5
```

Work out which plugin(s) the changes touch. Only bump the ones that actually
changed — a version bump on an untouched plugin forces a pointless reinstall for
everyone.

If the tree is dirty with unrelated user work, stop and ask. `claude plugin tag`
refuses to tag a dirty tree, and you should not sweep someone else's edits into a
release commit.

## 2. Pick the version

Semver, judged by what a *user of the plugin* experiences:

| Change | Bump |
|---|---|
| wording, a new example, a fixed typo in a skill | patch — `0.1.0` → `0.1.1` |
| a new skill, a new section, broader triggers | minor — `0.1.0` → `0.2.0` |
| a renamed or removed skill, a changed invocation, a new external dependency | major — `0.2.0` → `1.0.0` |

Renaming a skill directory is a breaking change even though nothing errors: the
user's `/plugin:old-name` stops existing.

## 3. Bump both manifests

The version lives in two files and they must agree:

- `plugins/<name>/.claude-plugin/plugin.json` → `version`
- `.claude-plugin/marketplace.json` → the entry for `<name>` → `version`

Edit both with a script, preserving indent and key order. Then prove they agree:

```sh
claude plugin tag ./plugins/<name> --dry-run
```

That is the cheapest check that exists — it validates the two manifests against
each other and prints the tag it would create. A mismatch fails here rather than
after the push.

If the description, keywords, or homepage changed too, mirror those into the
marketplace entry as well; they are duplicated by design and drift silently.

## 4. Validate

```sh
claude plugin validate .
claude plugin validate ./plugins/<name>
```

For anything more than a wording change, load it before shipping it:

```sh
claude --plugin-dir ./plugins/<name>
```

## 5. Commit, tag, push

```sh
git add -A
git commit -m "<name> <version>: <what changed, in a clause>"
claude plugin tag ./plugins/<name> --push -m "<name> %s"
git push
```

`claude plugin tag` creates `<name>--v<version>`, which is what keeps per-plugin
history legible in a repo holding several plugins. `%s` in the message expands to
the version. It refuses to overwrite an existing tag — if it does, the version
was already released and you need a higher one, not `--force`.

Update the docs alongside the code when the change is user-visible: the plugin
table in the repo README, and the marketplace's docs page if there is one — for
this repo, `plugins.md` in the `shadr.us` site repo. Commit and push that
separately.

## 6. Update the installed copy

```sh
claude plugin update <name>@<marketplace>
claude plugin details <name>@<marketplace>
```

`details` shows the version now installed — read it back and confirm it is the
version you just cut. The update takes effect in the next session; say so rather
than letting the user wonder why the old behavior persists in this one.

For a marketplace served from GitHub rather than a local path, run
`claude plugin marketplace update <marketplace>` first, or the update resolves
against a stale catalog.

## Rules

1. One number, two files. `claude plugin tag --dry-run` is how you prove it.
2. Bump only the plugins whose files actually changed.
3. Never `--force` a tag to reuse a version. Cut a new one.
4. Push before updating the installed copy — otherwise you install a version
   nobody else can get.
5. Iterating is `--plugin-dir`, not a release. Don't cut five versions to debug
   one skill.
6. Tell the user the new version is live next session, not this one.
