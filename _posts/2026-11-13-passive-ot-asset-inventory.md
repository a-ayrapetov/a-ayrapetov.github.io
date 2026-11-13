---
layout: post
title: "Inventory an OT network without sending it a single packet"
---
Every OT security program starts with the same question: what is actually on this network? On an office network the usual answer is a scan. On a network full of controllers, drives, HMIs and remote terminal units, a scan is the one answer you cannot use. Older controllers and protocol stacks can stop responding or fault when they are probed, and a scan during operations is a change nobody approved.

The good news is that the network already knows. Think of a hotel that needs a guest list. You could knock on every door, or you could read the key-card log at the front desk. The switches are the front desk.

## Four tables the switch already keeps

Every access switch builds these tables as a by-product of forwarding traffic. Reading them adds nothing to the OT endpoints.

1. **ARP table** on the layer 3 device: IP address to MAC address.
2. **MAC address table** on the access switch: MAC address to port and VLAN.
3. **Interface status**: port to VLAN, link state and the description someone typed years ago.
4. **CDP and LLDP neighbors**: which ports lead to other network gear, so you can leave them out of the endpoint list.

Join them on MAC address and port, and you get a list that says: this MAC, at this IP, sits on this port of this switch, in this VLAN, and the port says it is "Line 3 PLC". That is more useful than a scan result, because it tells you where the device physically is.

## What the join tells you that a scan cannot

- **An access port with several MAC addresses behind it** usually means an unmanaged switch, a hub, or a device with a small switch built in. Those are the places where segmentation quietly stops.
- **A MAC address with no ARP entry** is either a layer 2 only device or a device whose gateway lives somewhere you have not collected yet. Both are worth knowing.
- **A MAC that moves** between runs is either maintenance or something you need to ask about.

## Treat the existing inventory as a lead, not the truth

On one large industrial network the monitoring system's export was the official device list. In almost every row the device name was simply the IP address repeated, and the real name was buried in the first line of a notes field. Use whatever list exists to decide where to look first, then let the switch tables decide what is true.

## Collect like a guest, not an inspector

The collection itself has to be as quiet as the method.

- Read-only show commands on the switches only. Nothing goes to the OT devices. No ping sweeps, no scans on OT VLANs.
- Never poll safety systems, interlocks or serial-to-Ethernet converters, even with show commands.
- Throttle. A handful of concurrent sessions, a pause between commands, a pause between switches.
- Poll no more often than every 15 minutes. Hourly is a sensible default.
- Start with two switches you know well, check the result by hand, then scale.
- Keep every raw output next to the parsed result. When someone questions a line, the evidence is one file away.

## Three things that will catch you out

**ARP is per VRF.** On IOS-XE, `show ip arp` shows the global table. If the OT networks live in VRFs, collect `show ip arp vrf <name>` for each one, or half the IP column stays empty.

**Quiet devices age out.** A MAC address only stays in the table while the device keeps sending frames, and the default aging time on Catalyst switches is five minutes. A controller that talks once an hour may not be there when you look. Run the collection several times across a day and keep the union.

**Not every switch speaks Cisco.** Industrial switch vendors have their own CLIs, some with quirks that break automation. For those, read the same tables over SNMP from the standard bridge MIB instead of fighting the command line.

## Run it again and diff

A single inventory is a snapshot. The value comes from the second run. New MACs, MACs that vanished and MACs that moved ports are a change log for a network that is supposed to change rarely. A new MAC on a controller VLAN at two in the morning is the kind of event worth reading.

This is also where the frameworks point. The joint guidance CISA and its partners published in August 2025, [Foundations for OT Cybersecurity: Asset Inventory Guidance for Owners and Operators](https://www.cisa.gov/resources-tools/resources/foundations-ot-cybersecurity-asset-inventory-guidance-owners-and-operators), treats a maintained OT asset inventory as the base of a defensible architecture, and IEC 62443 zones and conduits cannot be drawn without one.

## The tool

I wrote [ot-asset-inventory](https://github.com/a-ayrapetov/ot-asset-inventory) to do the join. You give it a folder per switch with the saved show output, and it gives you a CSV of endpoints per port with flags for multi-MAC ports, MACs with no IP and locally administered addresses. Vendor names come from the IEEE OUI file if you supply it. A diff mode compares two runs and exits non-zero when something is new, gone or moved, so it can run on a schedule.

```
pip install git+https://github.com/a-ayrapetov/ot-asset-inventory
ot-inventory collect/ --oui oui.csv --csv inventory.csv
ot-inventory diff last-week.csv inventory.csv
```

It only reads text. It never connects to anything, which is the point.
