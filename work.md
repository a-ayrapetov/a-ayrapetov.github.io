---
layout: page
title: Work
permalink: /work/
---
Selected work, described without client names or client data. Each line is something I designed, built or led.

## OT and ICS security, large US transit agency

Design authority for identity, segmentation and detection on the operational networks of a large US transit agency, as a contractor.

- Segmentation designed under the TSA rail security directives, with hundreds of security zones along IEC 62443 across train control, SCADA, fire alarm, elevators, fare collection and CCTV.
- Asset discovery that found thousands of devices nobody had on record. The passive method I use for this: [Inventory an OT network without sending it a single packet]({% post_url 2026-11-13-passive-ot-asset-inventory %}).
- Closed-mode access control on industrial switches, being rolled out site by site with the blast radius held to one port. Method: [Closing access ports on industrial switches, one port at a time]({% post_url 2026-11-13-closed-mode-mab-one-port-at-a-time %}).
- Identity and audit platform on Cisco ISE, PKI, TACACS+ and Splunk, including an upgrade of a multi-node ISE cluster across two data centers. Lessons: [Moving switches to ISE TACACS+ without locking yourself out]({% post_url 2026-11-13-tacacs-migration-without-lockouts %}).
- Hardened more than 6,000 switches by automation and built the discovery, login and audit tooling the team uses; one night run reads over 9,000 devices.
- Monitoring on the train control network through TAPs feeding the OT and network detection platforms, installed in one overnight window with no service disruption.
- Flow analytics and NDR operated and tuned until detection was usable for the operations center. Method: [Cutting NDR alarm noise without going blind]({% post_url 2026-11-13-ndr-alarm-noise-without-going-blind %}).

## Campus fabric and access control, biotechnology company

- Took over a Catalyst Center managed SD-Access fabric and made it stable: certificate-based access control across eight virtual networks, ISE 3.2 to 3.4, all open vulnerabilities closed.
- Upgraded the fabric with zero rollbacks and moved the security groups and contracts from ISE to Catalyst Center group-based access control with zero failures.
- Proved a dedicated cloud circuit was saturated using controller interface history, which drove the upgrade to a larger circuit.
- Found the cause of a building-wide Wi-Fi outage and built a guarded provisioning script that restores service in seconds.
- Wrote the network security controls and evidence for the company's ISO 27001 certification.

## Firewall and WAN migration, state government agency

- FortiGate to Meraki MX across 17 sites (data centers, offices and branches) with no disruption, rebuilding site-to-site VPNs, RADIUS, BGP to Azure and web security.

## Voice platform refresh, county government

- Network and gateway lead for a Cisco call control and voicemail upgrade and a move from PRI to SIP on two CUBE gateways, built beside the live system. Read the whole county network by script and found a carrier outage nobody had reported.

## Data center refresh, higher education

- Lead architect for Nexus vPC top-of-rack pairs into Nexus 7700 cores under a live VMware vSAN stretched cluster. Caught a design versus purchase order mismatch and a license part that did not exist before the order went out.

## Hospital network security, 15 facilities

- Cisco ISE for more than 10,000 endpoints, including about 2,000 medical devices that cannot go down; ASA to Firepower under FMC for more than 30 firewalls; encrypted site-to-site VPN under HIPAA.

## Automation and AI

- Six open-source, read-only tools for ISE, Cisco configuration and Windows posture, plus the newer ones on the [tools page](/tools/).
- A knowledge platform for my practice built on Cloudflare Workers: a sealed container per client, versioned writes, an audit trail, secret scanning on every write and automatic checks on every deploy. How an AI assistant fits into production work: [An AI assistant on a production network that never gets a session]({% post_url 2026-11-13-ai-assistant-that-never-touches-the-network %}).
