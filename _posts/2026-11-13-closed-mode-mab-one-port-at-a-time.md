---
layout: post
title: "Closing access ports on industrial switches, one port at a time"
---
Closing access ports on an industrial network is like changing the locks in a building that never closes. Every door has to keep working for the people already inside while you work on it. The devices on these ports (cameras, intercoms, sensors, controllers, signs) mostly cannot run 802.1X, so they get MAC Authentication Bypass. In closed mode, a port passes nothing until the policy server says yes. One wrong group, one stale MAC address or one device that has not spoken yet, and something that matters goes dark.

This is the order of work I use for those rollouts with Cisco switches and Cisco ISE. I am using it on a rollout that spans hundreds of devices across many sites, one switch at a time.

## Agree the scope before touching a port

Run a short test with the owners on one port of each device type. Write down which types passed, which are approved, which are excluded and which are on hold. Anything tied to safety stays on hold until its owner approves in writing. That list is the contract for the rest of the rollout.

## Keep a safety net in policy

While ports are being closed, the catch-all authorization rule in ISE stays at permit. That is deliberately not the end state. It means a device that profiles oddly still gets on the network while you learn about it. Tightening the catch-all is a separate change, later, with its own approval.

## Trust no single device list

Before writing a single command for a site, build the device list from three sources and reconcile them: a fresh read-only capture of the MAC address tables from the switches, the endpoint export from ISE, and the tracking sheet the owners keep. Fix stale MACs first. Most outages in a rollout like this start with a wrong assumption about what is plugged in.

Then check ISE read-only: the site's switches exist as network devices, every endpoint sits in the right identity group, and the authorization rules for those groups exist.

## One switch at a time, always in the same order

1. Confirm the switch is a network device in ISE with the right shared secret.
2. Open a fresh admin session while the old one stays open. A working TACACS+ login proves nothing about RADIUS, so test each service on its own.
3. Pre-check that the switch reaches the policy servers and that the identity template is on the switch.
4. Apply the ports from one paste block with real names inline: a description that says what the device is, then the closed-mode template.
5. Test every port. Save only after the tests pass.

A port with several MAC addresses behind it needs its host mode chosen on purpose and agreed with the owner. Multi-host authenticates the first MAC and then opens the port for the rest. Multi-auth authenticates every MAC on its own. They are not interchangeable, and the wrong one either blocks the second device or lets in anything that plugs into the first.

## Never bounce a PoE port to retry

The reflex to shut and unshut a port to restart authentication is wrong here. On a PoE port, shut cuts power to the camera or intercom, which then has to boot again. Clear the session instead:

```
clear access-session interface GigabitEthernet1/0/12
```

On releases or configurations that still use the legacy form, the command is `clear authentication sessions interface ...`. Check with `?` on the switch.

## Quiet devices are not failed devices

MAB starts when the device sends its first frame. Some devices are quiet for minutes. No session and no MAC usually means the device has not said anything yet, so give it a few minutes and check link and power before you blame policy. A show command right after clearing a session can also come back empty for a few seconds. Run it again before you react.

And know whether ISE reacts to profile changes on your deployment. If a device's profile changes and ISE sends a change of authorization, a device that worked yesterday can drop today.

## What counts as done

Per port: `show access-session interface <port> details` shows the session authorized by MAB with the expected tag or VLAN, the device answers, and the ISE live log shows the pass. Per switch: the save is confirmed. Per site: every port in scope is on the tracking sheet as verified from output, not from memory.

Rollback is per port: remove the template and any host mode you added, then clear the session on that interface.

The full checklist is in my [closed-mode MAB playbook](https://github.com/a-ayrapetov/a-ayrapetov/blob/main/playbooks/closed-mode-mab-site-rollout.md). Before the rollout, the [passive inventory]({% post_url 2026-11-13-passive-ot-asset-inventory %}) is how the device list gets built without touching the devices.
