---
layout: post
title: "Moving switches to ISE TACACS+ without locking yourself out"
---
Moving switch logins from local accounts to Cisco ISE TACACS+ is routine work, and it is also the change most likely to lock you out of your own network. On one migration of older switches, a method list in the wrong shape cost a full day and three lockouts. The failure looked exactly like a bad password every time, which is why it took so long to see.

Think of a door that reads your keycard before it checks whether the card reader has power. If the reader is dead and nobody planned for that, you stand outside with a perfectly good card.

## The rule that decides whether your fallback works

Cisco IOS walks a method list in order, and it moves to the next method only when the current one gives no answer. Cisco's [authentication configuration guide](https://www.cisco.com/en/US/docs/ios-xml/ios/sec_usr_aaa/configuration/15-2mt/sec-cfg-authentifcn.html) spells it out: if the server does not respond, that is an error and the next method is tried; if the server answers with a rejection, the process stops there.

So `local` at the end of a list protects you from an outage of the policy servers. It does not protect you from a wrong password, a missing user on ISE, or a policy that says no. Those are answers, and an answer ends the login. Plan for both cases separately.

## A shape that survives a bad day

The pattern that held up across fleets of older and current switches:

```
aaa new-model
username breakglass privilege 15 secret <strong-local-secret>
!
aaa authentication login default local
aaa authentication login VTY_AUTH group ISE_TACACS local
aaa authorization exec VTY_AUTH group ISE_TACACS local if-authenticated
aaa accounting exec default start-stop group ISE_TACACS
aaa accounting commands 15 default start-stop group ISE_TACACS
!
ip tacacs source-interface Vlan100
!
line vty 0 15
 login authentication VTY_AUTH
 authorization exec VTY_AUTH
```

What matters in it:

- **A real local account with a secret**, so the fallback has someone to let in.
- **`local` after the server group** on login and exec authorization, and **`if-authenticated`** at the end of exec authorization, so an outage of the policy servers does not leave a user who already logged in without a shell. Prove that on one canary switch by taking ISE out of reach on purpose: how the list falls through for a user who exists only on ISE is worth seeing with your own eyes on your release.
- **The default login list stays local** and only the vty lines use the server list. The console keeps working the old way while you test.
- **Accounting to the server**, so you get a record of who typed what.

On some older IOS 15 releases the named `tacacs server` form parsed without complaint but never resolved the server address, and the legacy `tacacs-server host <ip> key <key>` form worked. That is a field observation, not something from a Cisco document, so check `show tacacs` after you configure it, whatever form you use.

## The source address has to match what ISE expects

ISE identifies a network device by the source address of its requests. `ip tacacs source-interface` pins that address. Without it, the switch uses whatever interface the route points out of, ISE sees an address it has no device entry for, and every login with an ISE account fails. A one digit typo in the ISE device entry does the same thing.

## The management VRF trap

The most confusing version of this failure happens when management lives in a VRF. The switch sends its TACACS+ requests from the global routing table, the replies never come back, and every login waits on a timeout of a few seconds before it falls back. With command authorization on, so does every command. `show tacacs` shows packets sent and none received.

The fix is a server group that is pinned to the VRF, as Cisco documents in [Per VRF for TACACS+ Servers](https://www.cisco.com/en/US/docs/ios-xml/ios/sec_usr_tacacs/configuration/15-1mt/Per_VRF_for_TACACS_Servers.html):

```
aaa group server tacacs+ ISE_TACACS
 server-private 192.0.2.10 key <key>
 server-private 192.0.2.11 key <key>
 ip vrf forwarding MGMT
 ip tacacs source-interface Vlan100
```

Prove reachability from the right table with `ping vrf MGMT 192.0.2.10 source Vlan100`. A plain ping from the global table proves nothing. While you are there, align logging, SNMP traps, NTP and NetFlow sources with the same VRF and interface.

## Before you bind the vty lines

1. Test the server group directly with `test aaa group ISE_TACACS <user> <password>`. Options differ by release, so check them with `?` on the device first. Count it as a pass only when the output literally says the user was authenticated.
2. Keep your current session open. Open a second session with the new login and prove it works before you close the first.
3. Change the vty lines and the console in separate windows. Never both at once.
4. Know your way back in before you start: a vty line still on local authentication, then taking the switch out of ISE so the server stops answering and `local` takes over, then the console, and a reload only if the bad state was never saved.

## Check the config before it checks you

I wrote [aaa-lockout-check](https://github.com/a-ayrapetov/aaa-lockout-check) to read a running-config and flag the shapes above before anyone binds a line: server lists with no fallback, console authorization that closes the last door, method lists or server groups that are referenced but never defined, a missing or dead source interface, and a source interface in a VRF that the server group does not use.

```
pip install git+https://github.com/a-ayrapetov/aaa-lockout-check
aaa-lockout-check "configs/*.cfg" --csv findings.csv
```

It reads text only and never connects to a device. Run it on the config you are about to push, not just the one that is running.
