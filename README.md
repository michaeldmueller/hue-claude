# hue-claude
Flash your Philips Hue lights when Claude Code finishes a turn or is waiting
on a permission.
- **Doesn't permanently change lights** - Uses the `signaling` API so that your lights return to their original state after flashing.
- **Local only** - Talks to the bridge over your LAN.
- **Quick setup** - Claude will guide you through initialization, which will require you to press the Hue button on your bridge once.

### Claude finished a task
https://github.com/user-attachments/assets/5e6b3a32-9b59-4852-acb1-62595adba8d0

### Claude needs your feedback
https://github.com/user-attachments/assets/fe07226a-1575-4c82-99a1-830d997108c9

## Install

Inside Claude Code:

```
/plugin marketplace add michaeldmueller/hue-claude
/plugin install hue@hue-claude
/hue:init
```

`/hue:init` will search for the bridge and prompt you to press the round link button on the top. After this, it will ask which lights you'd like to control (and write it to `~/.config/hue/config.env`). 

Or from the shell:

```
claude plugin marketplace add michaeldmueller/hue-claude
claude plugin install hue@hue-claude
```

### Requirements

- macOS or Linux. Bash 3.2 or later (the stock macOS `/bin/bash` works),
  `curl`, and `jq` (`brew install jq` / `apt install jq`).
- Bridge discovery uses `dns-sd` on macOS or `avahi-browse` on Linux
  (`apt install avahi-utils`). Without either it falls back to Hue's cloud
  lookup, or you can type the IP during `/hue:init`.
- Windows works through WSL or Git Bash, with the cloud lookup or a typed IP.
- A square Hue Bridge (v2) on firmware with the CLIP v2 API (2021 or later).
- Color bulbs that support signaling. `hue lights` shows which do.

## Customization

By default, this plugin will flash green to indicate a finished turn, and flash amber and red to indicate that Claude is waiting for your feedback. These defaults can be changed to any color or duration. Create a config file at `~/.config/hue/events.conf` (or tell Claude to) with your desired settings. Example: 

```
claude-done   on_off_color 2000 blue
claude-wait   alternating  8000 red  white
```

Signals are `alternating` (two colors), `on_off_color` (one), `on_off` (none),
and `breathe` (one smooth pulse in the current color, no duration or colors).

## License

MIT
