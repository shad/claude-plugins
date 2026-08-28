# cmux CLI inventory (captured from cmux 0.64.22)

Regenerate with `cmux --help`, `cmux <command> --help`, `cmux capabilities`.
`cmux docs [settings|shortcuts|api|browser|agents|dock|sidebars]` prints doc URLs
and curl commands for the current version.

## Targeting

Commands take `--window`, `--workspace`, `--pane`, `--surface`/`--tab`, each
accepting a UUID, a short ref (`workspace:2`), or an index. Output is refs by
default; `--id-format uuids|both` for UUIDs. Most commands accept `--json`.

Environment set in every cmux terminal:
`CMUX_WORKSPACE_ID`, `CMUX_SURFACE_ID`, `CMUX_TAB_ID`, `CMUX_SOCKET_PATH`.
`CMUX_QUIET=1` silences deprecation notices from legacy verb forms.

## Inspect
identify · tree [--all] · top [--processes] [--sort cpu|mem|proc] · memory ·
list-windows · current-window · workspace list · workspace-group list ·
list-panes · list-pane-surfaces · list-panels · surface-health ·
sidebar-state · list-status · list-log · list-notifications · list-buffers ·
capabilities · version · ping · debug-terminals

## Windows
new-window · focus-window · close-window · rename-window ·
move-workspace-to-window --workspace <w> --window <win>

## Workspaces
workspace create [--name --description --cwd --command --layout --group
  --group-placement afterCurrent|top|end --window --focus] ·
workspace list|close|rename|select|env|status|reconnect|disconnect ·
workspace loading on|off ·
reorder-workspace (--index|--before|--after) [--dry-run] ·
reorder-workspaces --order a,b,c [--dry-run] ·
workspace-action --action <pin|unpin|rename|clear-name|set-description|
  clear-description|move-up|move-down|move-top|close-others|close-above|
  close-below|mark-read|mark-unread|set-color|clear-color>
  [--title --color --description]

Colors: Red Crimson Orange Amber Olive Green Teal Aqua Blue Navy Indigo Purple
Magenta Rose Brown Charcoal, or #RRGGBB.

## Workspace groups
workspace-group list|create|ungroup|delete|rename|collapse|expand|pin|unpin ·
add --group <g> --workspace <w> · remove --workspace <w> ·
set-anchor · set-color --hex · set-icon --symbol <SF Symbol> ·
move (--to-index|--before|--after) · focus · new-workspace <g> [--placement]

The anchor workspace owns the group; its sidebar row *is* the group header.
`delete <g> --close-workspaces` is the only destructive form.

## Panes and surfaces
new-split <left|right|up|down> · new-pane [--type terminal|browser|simulator]
  [--direction] [--url] [--profile] · focus-pane · resize-pane (-L|-R|-U|-D)
  [--amount] · respawn-pane [--command] · pipe-pane --command ·
new-surface [--type terminal|browser|simulator|agent-session]
  [--provider codex|claude|opencode] · close-surface · move-surface
  [--pane --before --after --index] · split-off <dir> · reorder-surface ·
drag-surface-to-split · surface resume <set|show|get|clear> ·
refresh-surfaces · trigger-flash ·
tab-action --action <rename|clear-name|close-left|close-right|close-others|
  new-terminal-right|new-browser-right|move-to-new-workspace|reload|duplicate|
  pin|unpin|mark-unread|toggle-full-width-tab> [--title --url] ·
rename-tab <title> · move-tab-to-new-workspace

## Terminal I/O
send [--surface] <text>   (\n \r Enter, \t Tab) · send-key <key> ·
send-panel / send-key-panel --panel · read-screen [--scrollback --lines n] ·
capture-pane (tmux alias) · paste-buffer · list-buffers · display-message

## Sidebar reporting
set-status <key> <value> [--icon --color --priority] · clear-status <key> ·
set-progress <0.0-1.0> [--label] · clear-progress ·
log [--level --source] <msg> · list-log · clear-log · sidebar-state ·
notify --title [--subtitle --body] · list-notifications ·
mark-notification-read · dismiss-notification · open-notification ·
jump-to-unread · clear-notifications ·
right-sidebar <toggle|show|hide|focus|set|mode|files|find|vault|sessions|feed|dock> ·
sidebar <validate|reload|select|open> [name]

## Todo (the USER'S checklist — do not edit unprompted)
todo add|list|check|uncheck|start|edit|rm|clear|set|open

## Content panels
open <path-or-url>... · markdown [open] <path> ·
diff [patch|-] [--unstaged|--staged|--branch|--last-turn] [--base --cwd
  --layout split|unified --title --font-size]

## Remote
ssh / mosh / ssh-tmux / mosh-tmux <destination> [--transport --name --command
  --port --identity --forward-agent --ssh-option] ·
ssh-session-list|attach|cleanup · remote-daemon-status ·
remotes list|add|remove · vm (alias cloud) base|new|ls|status|snapshot|fork|
  restore|rm|exec|shell|ssh

## Browser (own subsystem — `cmux docs browser`)
browser open|open-split|goto|back|forward|reload · snapshot [--interactive]
· eval · wait · click|type|fill|press|select|scroll|hover · get <url|title|
text|html|value|attr|count|box|styles> · screenshot · console|errors ·
cookies|storage|profiles|tab|state · devtools|design-mode|focus-mode|zoom ·
react-grab

## Config and lifecycle
settings [open|path|docs|<target>] · config doctor|check|validate|path|paths|
  docs|reload · reload-config · themes list|set|clear · shortcuts ·
hooks setup|<agent> install|uninstall|event · hooks feed --source <agent> ·
agent-hibernation on|off · restore · restore-session · auth status|login|logout ·
enable-browser|disable-browser|browser-status · feed tui|clear ·
events [--after --cursor-file --name --category --reconnect --limit] ·
rpc <method> [json-params] · feedback

`cmux rpc` reaches methods with no CLI verb; `cmux capabilities` lists every
method name the running build exposes.

## Config file locations
- `~/.config/cmux/cmux.json` — cmux-owned settings (back up before editing)
- `~/.config/ghostty/config` — terminal rendering (font, theme, opacity, blur)
- `.cmux/dock.json` or `~/.config/cmux/dock.json` — right-sidebar buttons
- `~/.config/cmux/sidebars/*.swift` — custom sidebars (beta)
