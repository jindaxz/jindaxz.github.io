---
title: "Fixing AnyDesk lag with the lid closed on a 2017 MacBook Air running Linux"
date: 2026-10-23 16:45:00 -0700
description: "Diagnosing a Linux Mint XFCE display power setting that left AnyDesk without an active primary monitor when the MacBook Air lid was closed."
tags: [Linux, AnyDesk, XFCE, Troubleshooting]
---

AnyDesk became laggy when I closed this 2017 MacBook Air's lid, but felt normal with the lid open. The laptop runs Linux Mint 22.1 XFCE on X11, with an Intel HD Graphics 6000 display adapter.

## What was happening

The laptop was not suspending. XFCE's lid action was already set to do nothing, and systemd-logind was configured to ignore the lid switch. The log recorded the lid closing without a suspend event.

The display power state explained the lag. With the lid closed and the machine idle, X reported no active monitors and DPMS had switched the display off. AnyDesk's log then said it could not find a primary monitor and was falling back to capturing the desktop root.

## Changes made

I disabled XFCE display blanking and DPMS on both AC and battery, then restored the internal panel as the primary monitor in the current session:

```sh
xfconf-query -c xfce4-power-manager \
  -p /xfce4-power-manager/dpms-enabled --create -t bool -s false

for key in blank-on-ac blank-on-battery \
  dpms-on-ac-sleep dpms-on-ac-off \
  dpms-on-battery-sleep dpms-on-battery-off; do
  xfconf-query -c xfce4-power-manager \
    -p "/xfce4-power-manager/$key" --create -t int -s 0
done

xrandr --output eDP-1 --mode 1440x900 --primary
```

The XFCE settings are saved in the user's configuration. The `xrandr` command reactivated the internal display as the primary 1440×900 output in the running session. The lid remains set to do nothing, and the display brightness is at its lowest setting.

## Current result

After the change, the closed-lid session reported one active monitor, `eDP-1`, marked primary. I have not yet confirmed performance through another long AnyDesk session or after a reboot, so that still needs observation. If an existing AnyDesk connection continues to look stale, reconnecting should make it detect the active display again.

Related: [Making Wi-Fi recover automatically on this MacBook Air](/blog/macbook-air-wifi-stability/).
