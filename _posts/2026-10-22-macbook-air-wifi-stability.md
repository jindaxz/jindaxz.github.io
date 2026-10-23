---
title: "Making Wi-Fi recover automatically on a 2017 MacBook Air running Linux Mint"
date: 2026-10-22 12:00:00 -0700
description: "Troubleshooting Broadcom Wi-Fi on Linux Mint: driver updates, power settings, and a systemd timer to recover a stuck connection."
tags: [Linux, Networking, Troubleshooting]
---

A 2017 MacBook Air can still be useful as a Linux workstation, but its Broadcom Wi-Fi adapter relies on the proprietary `wl` driver. This machine is a `MacBookAir7,2` with a `BCM4360` adapter, running Linux Mint 22.1 and kernel 6.8.0-51.

The goal here was to reduce long Wi-Fi outages and make recovery automatic. The changes improve resilience; they cannot guarantee that Wi-Fi will never drop.

## What the logs showed

On October 13, NetworkManager recorded a link timeout and “ssid-not-found.” The computer stayed disconnected until the saved network was manually reconnected. That indicates the access point became unavailable to the adapter, but does not by itself prove whether the router, radio conditions, or driver caused it.

On October 19, the log entries were marked “user-requested” and came from the desktop Wi-Fi applet. By the time the machine was inspected, it had reconnected and NetworkManager reported full connectivity. That means the original failure was no longer present in the logs, so its exact cause could not be confirmed. No hardware failure was established.

## Changes made

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

## Current result and limits

At the latest check on October 22, the recovery timer was active and NetworkManager reported the Wi-Fi connection as connected with full connectivity.

This setup is meant to shorten a stuck outage and avoid a manual reconnect. It cannot repair an ISP or router outage, and it has not yet been observed through another long-running failure. The logs do not currently establish that the MacBook’s Wi-Fi hardware is defective.

If it drops again, capturing logs before toggling Wi-Fi will help identify the cause:

    nmcli device status
    nmcli networking connectivity
    journalctl -b -u NetworkManager --since "15 minutes ago" --no-pager
    journalctl -k -b --since "15 minutes ago" --no-pager | grep -Ei 'wl|broadcom|wifi|wlan|deauth|timeout|firmware'

For a separate issue with AnyDesk slowing down when the lid is closed, see [the display power-management fix](/blog/macbook-air-anydesk-lid-closed/).
