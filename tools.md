---
layout: page
title: Tools
permalink: /tools/
---
Open-source tools I wrote for my own engagements, rewritten so they carry no client and anyone can run them. Every one is read-only by default: show commands, GET calls or offline text parsing. Credentials are prompted at runtime or read from the environment and never written to disk. Every repository runs a secret scan and rejects private RFC 1918 addresses on each push.

| The question | Tool | What it does |
|---|---|---|
| What is on this OT network, without scanning it? | [ot-asset-inventory](https://github.com/a-ayrapetov/ot-asset-inventory) | Joins ARP, MAC, interface status and CDP/LLDP output into an endpoint list per switch port, flags hidden switches and MACs with no IP, and diffs runs for new, gone and moved devices |
| Will this TACACS+ change lock me out? | [aaa-lockout-check](https://github.com/a-ayrapetov/aaa-lockout-check) | Reads a running-config and flags AAA shapes that lock you out: no fallback, console authorization, undefined lists, a dead source interface, a management VRF mismatch |
| Why did this endpoint land in the wrong place? | [ise-endpoint-explain](https://github.com/a-ayrapetov/ise-endpoint-explain) | The live ISE session plus an authentication timeline: switch and port, authorization profile, SGT, matched rule and every failure reason in plain English |
| Is ISE healthy before tomorrow's change? | [ise-health-audit](https://github.com/a-ayrapetov/ise-health-audit) | Nodes down, missing admin high availability, system and trusted certificates expired or expiring within 30 and 60 days |
| Did anyone change the switches outside change control? | [cisco-config-drift](https://github.com/a-ayrapetov/cisco-config-drift) | Baselines every running-config and diffs anything that changed |
| Which ports need attention first? | [cisco-interface-health](https://github.com/a-ayrapetov/cisco-interface-health) | Parses `show interfaces` and ranks ports by urgency: err-disabled, CRC, duplex mismatch, drops, resets, each with the likely cause |
| How hardened is this fleet? | [ios-hardening-audit](https://github.com/a-ayrapetov/ios-hardening-audit) | Every hardening gap per device with severity, the evidence line and the exact fix |
| Is this Windows machine ready for work? | [win-security-posture-audit](https://github.com/a-ayrapetov/win-security-posture-audit) | TPM, Secure Boot, Defender, BitLocker, firewall, startup items and drivers in one report, plus a fix plan it does not apply |

The method behind them is in the [playbooks](https://github.com/a-ayrapetov/a-ayrapetov#how-i-work-playbooks).
