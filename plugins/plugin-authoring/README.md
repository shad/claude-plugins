# plugin-authoring

Two skills for maintaining a plugin marketplace repo like this one.

## Install

```
/plugin marketplace add shad/claude-plugins
/plugin install plugin-authoring@shad
```

## Skills

### `new-plugin`

Takes something that already works locally — a skill in `~/.claude/skills/`, an
agent, a hook — and makes it installable: creates the plugin directory and
manifest, adds the `marketplace.json` entry, validates, documents it in the
README and docs page, commits, pushes, and installs it.

It also handles the parts that are easy to forget: keeping the version in the
two manifests in sync, grepping the skill for machine-specific paths before it
goes public, and retiring the original from `~/.claude/skills/` so you don't end
up loading two copies of the same instructions.

### `release-plugin`

Ships a change to a plugin you've already published: bumps the version in both
manifests, validates, tags `<name>--v<version>`, pushes, and updates your
installed copy.

Both are model-invoked — "turn this skill into a plugin", "ship the cmux change"
— or callable directly as `/plugin-authoring:new-plugin`.

## Finding the marketplace

Both skills locate the marketplace repo by reading
`~/.claude/plugins/known_marketplaces.json` for a `"source": "directory"` entry,
so they work from any project, not just from inside the marketplace repo.
