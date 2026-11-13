---
layout: post
title: "Cutting NDR alarm noise without going blind"
---
A noisy detection platform is a smoke alarm that goes off every time someone makes toast. After a week nobody looks up, and the one real fire gets the same shrug. Flow analytics and network detection platforms on large networks often raise thousands of alarms a day. Most of them come from a handful of rules and a handful of hosts nobody ever classified.

The tempting fix is to switch the loud rules off. The dashboard goes quiet and the network goes invisible. Here is the loop I use instead. It works with any flow analytics or NDR platform that has host groups and custom rules, and a team of two or three can run it a few items a day.

## Measure hosts, not alarms

Count **hosts in alarm per day** for a rule, not raw alarms. Raw counts swing wildly with traffic volume and reward the wrong fixes: suppress one chatty host and the count drops by thousands while nothing about your visibility improved.

## The loop

1. **Size it.** Pick the rule with the most hosts in alarm. Record seven daily values. That is your before number. Skip anything small.
2. **Find who.** List the sources and targets for that rule. One target repeated across many sources is usually one cause.
3. **Prove what it is.** Host report, flow search, reverse DNS, the asset inventory. A host nobody can classify is a finding in its own right.
4. **Decide.** It is noise only if you can name the owner and the reason. Answers from the internet, traffic that crosses zones, scanning by unknown hosts, and anything without an owner stay as alarms and get reported.
5. **Change one thing.** Read the rule's existing exclusions first. Take a screenshot of the rule before editing: that picture is your rollback. Prefer putting the host into an existing functional group that the rule already excludes. If the rule does not exclude that group, classify the host and add the exclusion. Narrowest scope, one rule per change.
6. **Log and prove.** One row per change in a shared tracker: before, after, where it was applied, how to undo it.

## Proof that the change worked

- After the save time, the device still sends the same traffic (a flow search shows it), and no new alarm for that rule starts after the save time. Traffic continues and no new alarm: that is the proof.
- The rule now has exactly one more exclusion than before. Fewer means something was dropped during the edit.
- The next day's hosts in alarm is lower than the before number. Fixes for overnight scanners can only be proven the next morning.
- Run the same check from an export or an API script, so the numbers do not depend on who looked at the dashboard.

## Traps

- Adding a host to a group does nothing if the rule does not exclude that group. Read the exclusions first.
- Some rule editors drop existing exclusions while you edit. Count them before and after.
- Old alarms can stay marked active after the fix. Judge by when an alarm first became active.
- Group names go stale. Check what is actually in a group before tuning around it.
- Scanners left unclassified make quiet devices look like they are breaking segmentation, simply because they answer the scan.
- Safety-related systems never go into an exception group. Report them to their owner instead.
- Temporary exception groups need a written membership rule, an owner and an exit date.

## Why it works

Most of the noise comes from hosts that were never put where they belong. Classifying them fixes many rules at once, and what is left on the dashboard starts to mean something again. A week after each change, the before and after numbers are in the tracker, and nothing that was excluded has an unknown owner.

The same loop, written as a checklist, is in my [NDR alarm noise playbook](https://github.com/a-ayrapetov/a-ayrapetov/blob/main/playbooks/ndr-alarm-noise-reduction-loop.md). Classifying hosts is far easier with a current device list, which is what the [passive OT inventory]({% post_url 2026-11-13-passive-ot-asset-inventory %}) is for.
