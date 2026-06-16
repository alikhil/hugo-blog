---
title: "Control Monitor Input Source on macOS with BetterDisplay and DDC"
date: 2024-10-16T20:31:06+03:00
lastMod: 2026-06-16T20:31:06+03:00
description: "Learn how to switch monitor input sources on macOS using BetterDisplay, DDC/CI, and CLI commands. Configure keyboard hotkeys via the paid app or free Raycast scripts."
categories:

- mac

tags:

- tutorial
- tips
- mac

---

If you as me have single monitor and 2 Mac devices (for example, I have corporate Macbook and personal Mac Mini) you may want to use the same monitor for both devices. And you may want to switch between them without unplugging and plugging cables or selecting input source using monitor buttons.

In this post I will show you how to configure hotkeys for that.

<!--more-->

> **Quick answer**
>
> - **App**: Use [BetterDisplay](https://github.com/waydabber/BetterDisplay) — it exposes a CLI and GUI to switch monitor inputs via DDC/CI.
> - **When DDC/CI works**: Your monitor must support the DDC/CI protocol and have it enabled. Most modern displays (Dell, LG, BenQ, etc.) support it.
> - **When it won't work**: Some monitors, USB-C hubs, or cheap HDMI adapters block DDC signals. If BetterDisplay shows no DDC Input Sources, your hardware path doesn't pass DDC — try a direct cable instead of a hub or adapter.

## Hardware

You will need a monitor with **multiple input sources**.
For example, I have Dell S2722QC tt has 2 HDMI ports and 1 USB-C port where:

* Macbook Air connected to port HDMI-2
* Mac Mini connected to port USB-C-1

## Software

There is app called [BetterDisplay](https://github.com/waydabber/BetterDisplay) that has a lot of powerful features. But for our case we need only one feature - **change display inputs using DDC**.

Install it on both Macs. You will have 14 days trial period with all PRO features.

**Enable Accessibility for BetterDisplay in System Settings -> Privacy & Security -> Accessibility.**

Then try to switch input source by clicking on BetterDisplay icon in the menu bar -> *DDC Input Source -> Select next port.*

|   |  |
|-----------|--------------|
| ![Menu Shot](/images/posts/menu-shot.png) | ![Input Choose](/images/posts/input-choose.png) |

If it works, you can continue to the next step.
Otherwise check if your monitor supports DDC protocol and ensure Accessibility is enabled for BetterDisplay.

### Paid option

If you are ready to pay 19$/19€ x2 for both Macs you can [buy BetterDisplay](https://betterdisplay.pro/buy/). And then configure hotkeys in the app settings *Settings -> Keyboards -> Custom keyboard shortcuts -> DDC Input Source*.

Click "Record Shortcut" and press the key combination you want to use, for example `CMD + F1` and `CMD + F2`.

![alt text](/images/posts/shortcuts.png)


### Free option

If you like me don't want to pay for 40$ for single feature there is a hacky way to do it.

We need an app that can handle hotkeys and run shell commands. I use [Raycast](https://www.raycast.com/), so called "Spotlight on steroids" and it can handle custom hotkeys. Or you can use any other app you like.

#### BetterDisplay CLI commands

BetterDisplay ships a CLI you can call from any script or terminal. Here are the most useful commands:

```bash
# List all detected monitors and their current input source
/Applications/BetterDisplay.app/Contents/MacOS/BetterDisplay get -vcp=inputSelect

# Switch to a specific input by DDC value (replace 18 with your value)
/Applications/BetterDisplay.app/Contents/MacOS/BetterDisplay set -ddc=18 -vcp=inputSelect

# Cycle to the next available input source
/Applications/BetterDisplay.app/Contents/MacOS/BetterDisplay set -nextDDCInput -vcp=inputSelect

# Target a specific monitor by display index (0-based) when you have several
/Applications/BetterDisplay.app/Contents/MacOS/BetterDisplay set -ddc=18 -vcp=inputSelect -displayIndex=0
```

The DDC value (`-ddc=`) maps to the physical port. You can find the right values in *Settings -> Displays -> "Your monitor name" -> DDC Input Sources* (see the **Value** column).

#### Configuring shell command

Before configuring **Raycast** we need to know `ddc` value for each input source. To do so, go to *Settings -> Displays -> "Your monitor name" -> DDC Input Sources*, and save IDs from **Value** column for each input source:

![alt text](/images/posts/ddc-input-sources.png)

In my case it's `18` for HDMI-2 and `25` for USB-C-1.

Then create a directory `~/raycast-scripts` and put there a script `change-input-source.sh`:

```bash
#!/bin/bash

# See full documentation here: https://github.com/raycast/script-commands
#
# Required parameters:
# @raycast.schemaVersion 1
# @raycast.title Switch Monitor Input Source
# @raycast.mode silent
#
# Optional parameters:
# @raycast.icon 🖥️
# @raycast.packageName Raycast Scripts

DEST_ID=18 <PUT YOUR DDC ID HERE>
DEST_NAME="Home Mac Mini"

# PUT YOUR ACTUAL HOSTNAME
if [ `hostname` == "Aliks-Mac-mini.local" ]; then
  DEST_ID=25 <PUT YOUR SECOND DDC ID HERE>
  DEST_NAME="Work Macbook Air"
fi

echo "Switching monitor input source to $DEST_NAME"
/Applications/BetterDisplay.app/Contents/MacOS/BetterDisplay set -ddc=$DEST_ID -vcp=inputSelect
```

After that try to run it in terminal:

```bash
chmod +x ~/raycast-scripts/change-input-source.sh
~/raycast-scripts/change-input-source.sh
```

If it works, you can continue to the next step.

#### Configuring Raycast

1. Open Raycast and go to *Settings -> Extensions -> Search for Scripts*
2. Click on *Add Script Directory* and select `~/raycast-scripts`
3. Click on *Record Shortcut* for newly added script and press the key combination you want to use, for example `CMD + F1` on first Mac and `CMD + F2` on second.

![alt text](/images/posts/raycast-settings.png)


And that's it! Now you can switch input source using hotkeys.
