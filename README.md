# Claude Code plugins

A [plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces) of
skills and agents I use daily. Documented at [shadr.us/plugins](https://shadr.us/plugins).

## Install

```
/plugin marketplace add shad/claude-plugins
/plugin install cmux-control@shad
```

## Plugins

| Plugin | Description |
| --- | --- |
| [cmux-control](plugins/cmux-control) | Drive the cmux terminal from inside a session. Groups sidebar rows by git worktree, carries PR state on pills, and defines what every channel means. |
| [plugin-authoring](plugins/plugin-authoring) | Package a skill or agent into this marketplace and cut releases from it — manifests, validation, docs, tag, install. |

## Layout

```
.claude-plugin/marketplace.json   the catalog
plugins/<name>/                   one directory per plugin
  .claude-plugin/plugin.json      the plugin manifest
  skills/<name>/SKILL.md          skills
```

Adding a plugin means adding a directory under `plugins/` and an entry in
`marketplace.json`.

## Development

```
claude plugin validate .                       # validate the marketplace
claude --plugin-dir ./plugins/cmux-control     # load a plugin without installing
```

## License

Apache License 2.0. See [LICENSE](LICENSE).
