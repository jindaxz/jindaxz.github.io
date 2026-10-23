---
title: "A 2017 MacBook Air as a Linux workstation for automated work"
date: 2026-10-22 12:00:00 -0700
last_modified_at: 2026-10-23 17:00:00 -0700
description: "Repurposing a MacBook Air with Linux Mint: automatic Wi-Fi recovery and a display power fix for remote access with the lid closed."
tags: [Linux, Automation, Networking, Troubleshooting]
---

I’m using a 2017 MacBook Air as a Linux workstation for automated work, with remote access through AnyDesk. For that setup, I want the machine to stay reachable and remain usable with the lid closed. Two issues needed attention: Wi-Fi that could stay disconnected after a drop, and remote desktop lag when the display powered off.

These are my setup and troubleshooting notes from October 22–23, 2026. The changes are in place; long-running reliability still needs observation.

## The workstation

The machine is a `MacBookAir7,2`, running Linux Mint 22.1 XFCE on X11 with kernel `6.8.0-51`. It has a Broadcom `BCM4360` Wi-Fi adapter using the proprietary `wl` driver, and Intel HD Graphics 6000. The lid action is set to do nothing so closing it does not suspend the workstation.

## Wi-Fi recovery

The goal was to reduce long Wi-Fi outages and recover automatically when the connection gets stuck.

### What the logs showed

On October 13, NetworkManager recorded a link timeout and “ssid-not-found.” The computer stayed disconnected until the saved network was manually reconnected. That indicates the access point became unavailable to the adapter, but does not by itself prove whether the router, radio conditions, or driver caused it.

On October 19, the log entries were marked “user-requested” and came from the desktop Wi-Fi applet. By the time the machine was inspected, it had reconnected and NetworkManager reported full connectivity. That means the original failure was no longer present in the logs, so its exact cause could not be confirmed. No hardware failure was established.

### Keep Wi-Fi power saving disabled

NetworkManager already had Wi-Fi power saving disabled globally. I also set it explicitly on the saved home-network profile, so the profile continues to use the disabled setting.

### Update and reload the Broadcom driver

The installed Broadcom package was behind the available Ubuntu update. I upgraded `bcmwl-kernel-source` and `broadcom-sta-dkms` to version `6.30.223.271+bdcom-23ubuntu1.3`, rebuilt the module for the installed kernel, and reloaded `wl`. The update includes a fix for an array-bounds warning that had appeared when the older driver loaded.

### Allow either Wi-Fi band

I tested band-specific settings, then removed the 5 GHz-only restriction. The saved profile can now use either 2.4 GHz or 5 GHz, so it is not stranded if one band disappears. Automatic connection remains enabled, with unlimited retries.

### Add automatic recovery

A systemd timer checks connectivity every two minutes. After three consecutive failed checks, it reconnects the saved Wi-Fi profile. If NetworkManager cannot reconnect, it reloads the Broadcom driver and tries again. It waits at least 15 minutes between recovery attempts and does nothing when Wi-Fi has been switched off.

The timer is `wifi-autorecover.timer`; its script is `/usr/local/sbin/wifi-autorecover`. To inspect it:

    systemctl status wifi-autorecover.timer
    journalctl -t wifi-autorecover

### Result and limits

At the latest check on October 22, the recovery timer was active and NetworkManager reported the Wi-Fi connection as connected with full connectivity.

This setup is meant to shorten a stuck outage and avoid a manual reconnect. It cannot repair an ISP or router outage, and it has not yet been observed through another long-running failure. The logs do not currently establish that the MacBook’s Wi-Fi hardware is defective.

If it drops again, capturing logs before toggling Wi-Fi will help identify the cause:

    nmcli device status
    nmcli networking connectivity
    journalctl -b -u NetworkManager --since "15 minutes ago" --no-pager
    journalctl -k -b --since "15 minutes ago" --no-pager | grep -Ei 'wl|broadcom|wifi|wlan|deauth|timeout|firmware'


## Remote access with the lid closed

AnyDesk became laggy when I closed this 2017 MacBook Air's lid, but felt normal with the lid open. The laptop runs Linux Mint 22.1 XFCE on X11, with an Intel HD Graphics 6000 display adapter.

### What was happening

The laptop was not suspending. XFCE's lid action was already set to do nothing, and systemd-logind was configured to ignore the lid switch. The log recorded the lid closing without a suspend event.

The display power state explained the lag. With the lid closed and the machine idle, X reported no active monitors and DPMS had switched the display off. AnyDesk's log then said it could not find a primary monitor and was falling back to capturing the desktop root.

### Display settings

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

### Result and limits

After the change, the closed-lid session reported one active monitor, `eDP-1`, marked primary. I have not yet confirmed performance through another long AnyDesk session or after a reboot, so that still needs observation. If an existing AnyDesk connection continues to look stale, reconnecting should make it detect the active display again.


## Where the setup stands

The Wi-Fi recovery timer is active, and the latest network check reported full connectivity. After the display changes, the closed-lid session also reported an active primary monitor. Together, these changes address the two problems I encountered while preparing this MacBook Air for unattended work.

The next check is how it behaves through a reboot and an extended period of automated work and remote access. The recovery timer can help with a stuck Wi-Fi connection, but it cannot resolve an ISP or router outage. The display fix also needs a longer AnyDesk session to confirm that the lag stays resolved.
