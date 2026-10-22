# hue-claude

Flash your Philips Hue lights when Claude Code finishes a turn or is waiting
on a permission. Look up from the other screen, see green, come back.

- **Signaling, not state.** Uses the Hue bridge's `signaling` API: the bulb
  flashes for a few seconds and restores itself. Nothing is saved, nothing can
  be left in the wrong state.
- **Local only.** Talks to the bridge over your LAN. No cloud, no account.
- **No runtime.** One bash script plus `jq`. No Python, no Node.
- **Events, not colors.** Hooks name an event (`claude-done`), the script maps
  it to an effect. Change colors in one place, or add your own triggers.

## Install

Inside Claude Code:

```
/plugin marketplace add michaeldmueller/hue-claude
/plugin install hue@hue-claude
/hue:init
```

`/hue:init` finds the bridge, has you press its link button, lists your
lights, and writes the choice to `~/.config/hue/config.env`. From then on
every session flashes green when Claude stops and amber/red while a
permission prompt is waiting.

Or from the shell:

```
claude plugin marketplace add michaeldmueller/hue-claude
claude plugin install hue@hue-claude
```

### Requirements

- macOS. The script uses `dns-sd`, `security`, and the stock
  `/bin/bash` 3.2.
- `jq` (`brew install jq`) and `curl`.
- A square Hue Bridge (v2) on firmware with the CLIP v2 API (2021 or later).
- Color bulbs that support signaling. `hue lights` shows which do.

## What it does

| Hook | Event | Default effect |
| :- | :- | :- |
| `Stop` | `claude-done` | green, 3 s |
| `Notification` (`permission_prompt`) | `claude-wait` | amber/red alternating, 5 s |

`Stop` fires after every response, including short ones. `Notification` is
matched on `permission_prompt` only, so amber means Claude is blocked on an
approval. Idle timeouts are deliberately not hooked: they fire whether Claude
asked you something or you simply walked away.

Hooks run async so the bridge call never delays Claude, and the script always
exits 0 so a dead bridge can never block a turn. Each hook payload is appended
to `~/.config/hue/logs/hooks.log` so a surprise flash can be traced to a session.

## Customizing

**Colors and durations.** Create `~/.config/hue/events.conf`. One event per
line: `name signal duration_ms color [color2]`. It overrides the built-ins by
name:

```
claude-done   on_off_color 2000 blue
claude-wait   alternating  8000 red  white
build-failed  on_off_color 3000 red       # your own trigger
```

Signals are `alternating` (two colors), `on_off_color` (one), `on_off` (none).
Colors are `red green darkgreen blue amber purple white`. `hue events` lists everything.

**Your own triggers.** Anything that can run a command can flash the lights:

```
hue event build-failed
```

Add the event to `events.conf` and call it from a Makefile, a cron job, a git
hook, a Shortcut. `hue` is on PATH inside Claude Code's shell; elsewhere use
the full path, which `hue config show` prints.

**Group mode.** `hue config set GROUP_ID <id>` (ids from `hue groups`) sends one
request to a room or zone instead of one per bulb, so every light switches in
the same instant. `LIGHT_IDS` is ignored while it is set.

**Keychain.** `hue pair --keychain` stores the app key in the macOS Keychain
instead of the config file.

**TLS.** The bridge presents a self-signed certificate, so requests use
`curl -k` by default. To pin instead, set `HUE_CACERT` in `config.env` to the
path of the Hue root CA.

## Commands

```
hue event <name>                run a named event (always exits 0)
hue events                      list events and their effects
hue signal <sig> <ms> <c> [c2]  raw signal on the configured target
hue init [--keychain]           guided terminal setup (use /hue:init inside Claude Code)
hue pair [--keychain] [--no-wait]
hue discover                    bridge IP(s) via mDNS
hue lights                      lights with id, name, room, signal_values
hue groups                      rooms/zones with their grouped_light id
hue test                        reachability + auth check
hue config [show|path|set K V]  inspect or edit config.env
```

Config directory defaults to `~/.config/hue`; override with `HUE_CONFIG_DIR`.

## Without the plugin

The script works on its own. Clone the repo, put `bin/hue` on your PATH, run
`hue init`, then add the hooks from `hooks/hooks.json` to
`~/.claude/settings.json` with the full script path in place of
`${CLAUDE_PLUGIN_ROOT}/bin/hue`.

## Why not the others

There are a few similar projects. Most are Python, set light state directly
and then have to save and restore it, and install by editing your settings
file. This one is a plugin, has no runtime, and uses signaling so the bulb
restores itself. It does not show a persistent "busy" color while Claude
works, and it does not detect prose questions. If you want those,
[claude-code-hue](https://github.com/Minetorpia/claude-code-hue) and
[claude-hue-notify](https://github.com/oguzhanT/claude-hue-notify) do them.

## License

MIT
