---
layout: post
title: "An AI assistant on a production network that never gets a session"
---
I use an AI assistant on almost every engagement now: switch and controller work, ISE policy, monitoring platforms, industrial networks. It has never once held a session on a client device, and that is the reason it is safe to use at all.

Think of a surgeon and a scrub nurse who knows every procedure by heart. The nurse prepares, checks and hands over the right instrument at the right moment. Only the surgeon touches the patient. An assistant on a production network works best in the same split: it writes the commands, scripts and click paths faster than any person, and a person runs them.

## What goes wrong without rules

The failures are predictable, and none of them are exotic:

- Commands written for a different software release than the one running.
- "Done" or "fixed" reported with no output to back it.
- The same context explained again in every new chat.
- Work that stops whenever the one person who remembers the details is away.

Each rule below exists because one of these happened.

## The rules

**Split the roles.** The assistant has no session on any device or console. It writes; a person runs, then hands back the output as a log file, an export or a screenshot. In regulated and availability-first environments this is also the only arrangement a change board will accept.

**Keep one record.** One knowledge base per engagement holds the rules, the facts, a status board and a tracker. Every chat reads it first and writes back what changed in the same reply. A chat has no reliable end, so nothing is saved for later. Anyone, or any other assistant, pointed at the record can pick up the work.

**Source every fact that can change.** Addresses, VLANs, tags, versions, CLI syntax and GUI paths come from output in the conversation, from the record, or from the vendor document for the exact platform and release. Anything else is marked as unverified, with the command that would settle it. Never from the model's memory, because the model's memory is where commands from the wrong release come from.

**Climb a fixed ladder for missing facts.** Output already given, then the record, then local files, then vendor documentation, then one read-only command for the person to run. The assistant climbs the ladder before it asks a question.

**No completion claim without evidence.** "Done" requires output read in that same reply. A colleague saying it worked, a plan, or a script that exists does not count. A scheduled job finished only if its final status line is in the log.

**Ship every change in one shape.** Scope, operational impact, the verification command and the value it should return, a rollback ready to paste, and how the device is reached if the change locks you out. One change at a time, a canary first, stop at the first device that does not verify.

**Scripts are read-only unless asked.** A change script has a dry run, an explicit apply switch, a backup before, a verification after and a rollback file. Credentials come from the operating system's credential store, never from the script, a log or the chat.

**Keep a fallback.** The record is plain text behind a standard protocol and mirrored to a folder. If one assistant is unavailable, another reads the same files. If none is, the runbooks are complete enough for a person.

## Why the split works

An assistant is strongest at recall and drafting and weakest at knowing what is true on a live network right now. The split puts each side where it is reliable: the person supplies ground truth, the assistant supplies speed, and the record carries memory across sessions and people.

The test I use is simple. Pick any statement the assistant made today and ask where it came from. If the answer is a path in the record, a line of output, or a vendor page for the right release, the method is working. If the answer is "it just knew", it is not.

## Where to find more

Two of my playbooks carry the method further: [gated production changes](https://github.com/a-ayrapetov/a-ayrapetov/blob/main/playbooks/gated-production-change.md) and [the script contract](https://github.com/a-ayrapetov/a-ayrapetov/blob/main/playbooks/script-contract.md). The gates and proofs those describe are [code with tests](https://github.com/a-ayrapetov/a-ayrapetov/blob/main/playbooks/templates/netops_gates_and_proofs.py).
