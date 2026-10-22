---
name: init
description: Set up the Hue plugin - pair with the Philips Hue bridge, choose which lights flash, and test. Run this once after installing the plugin, or again to change lights.
disable-model-invocation: true
---

# Hue setup

You are walking the user through pairing Claude Code with their Philips Hue
bridge. The `hue` script does all the work; you run its commands and relay
results. It is on PATH while this plugin is enabled. If `hue` is not found,
use `"${CLAUDE_PLUGIN_ROOT}/bin/hue"` instead.

Config is written to `~/.config/hue/config.env`. Never print the app key.

## Steps

1. **Check current state.** Run `hue test`. If it prints a bridge name and the
   target has lights, say so and ask whether they want to re-pair or just
   change lights. If they are happy, skip to step 6.

2. **Find the bridge.** If `hue config show` has no `BRIDGE_IP`, run
   `hue discover`. One address: `hue config set BRIDGE_IP <ip>`. Several:
   ask which. None: ask the user for the IP (it is shown in the Hue app under
   Settings > Bridges) and set it.

3. **Pair.** Tell the user: "Press the round link button on top of the Hue
   bridge, then tell me when it's done." Wait for their reply. Then run
   `hue pair --no-wait`. On "link button was not pressed", ask them to press
   it and try once more. The pair command writes the key itself.

4. **Whole room or specific lights?** Run `hue groups` and show the rooms
   and zones with their light counts. Ask whether they want a whole room or
   zone to flash, or only certain lights. Most people want their office.

5. **Set the target.**
   - *Room or zone:* `hue config set GROUP_ID <id>` with the GROUP_ID from
     the table. One request to the group makes every bulb switch at the same
     instant. Nothing else is needed; `LIGHT_IDS` is ignored while a group
     is set.
   - *Specific lights:* run `hue lights`. It shows each light's name, its
     room, and its signal values. Lights without any signal values cannot
     flash and should not be offered. Ask which to use, then run
     `hue config set LIGHT_IDS "<id> <id> ..."` with the chosen IDs, space
     separated. If a group was set earlier, clear it with
     `hue config set GROUP_ID ""`.
   Do not query the bridge API directly; the two commands above give
   everything needed.

6. **Test.** Run `hue test`, then `hue event claude-done` and ask whether the
   lights flashed green. If not, check `~/.config/hue/logs/hooks.log` and the
   command's stderr for the bridge error and fix it.

7. **Finish.** Tell them:
   - Green flash = Claude finished a turn. Amber/red = Claude is waiting on a
     permission. Both are already active for every session.
   - `hue events` lists the effects. Custom events or different colors go in
     `~/.config/hue/events.conf`, one per line, for example
     `claude-done on_off_color 2000 blue`.
   - `/hooks` shows the registered hooks.
