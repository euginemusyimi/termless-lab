# Lab Task Debrief — VLAN tag mismatch on Proxmox VM NICs (recurring, 3 incidents)

## 0. Task identifier

**Date:** 2026-10-11 (latest incident); consolidates two earlier incidents from the MSP homelab build
**Task/lab name:** VM lands on the wrong VLAN (or none) after Proxmox changes — DC01, client VM, netbird-router
**Related project:** MSP Homelab (Part 1) / NetBird routing peer (Part 1 VPN row, Part 6 Phase 8)
**Time spent:** (fill in — the netbird-router incident alone spanned Oct 9 to Oct 11 across sessions)

---

## 1. What was I trying to do?

Keep every lab VM on its intended VLAN (DC01 on 20, domain client on 30, netbird-router on 20) so each one can reach its OPNsense gateway and the rest of the network.

---

## 2. What did I actually do?

Three separate occurrences, same fix each time.

1. **DC01** — after a Proxmox hardware change/restart it appeared on `192.168.0.x` (the flat home network) instead of VLAN 20. Diagnosed with `Get-NetIPAddress` and traceroute. Fixed in Proxmox → VM → Hardware → Network Device (correct bridge + VLAN tag), then a full Stop → Start. Knock-on effect: DC01 then took `10.10.20.100` from Kea DHCP instead of its static `10.10.20.10`; re-applied the static IP with `New-NetIPAddress` / `Remove-NetIPAddress`, reset DNS to `127.0.0.1`, re-added the forwarder, ran `ipconfig /registerdns` and `Restart-Service Netlogon`.
2. **Domain client VM** — appeared on VLAN 40 instead of VLAN 30. Same Proxmox NIC fix, same full Stop → Start.
3. **netbird-router** (Oct 9–11) — worked on Sep 27, then 12 days later was unreachable from its gateway. Fixed by setting the VLAN tag to 20 on its NIC in Proxmox. (The exact prior tag value wasn't recorded; fill in if known.)

```
# netbird-router diagnostics that pointed at layer 2
ping -c 2 10.10.20.1      -> From 10.10.20.50 icmp_seq=1 Destination Host Unreachable
ping -c 2 1.1.1.1         -> Destination Host Unreachable
ping -c 2 api.netbird.io  -> Temporary failure in name resolution
sudo netbird status       -> Management: Disconnected ... dial context: context deadline exceeded
# fix: Proxmox -> netbird-router -> Hardware -> Network Device -> VLAN Tag = 20
```

---

## 3. What broke — on purpose or not?

**Broke on purpose? (Y/N):** N

**What was the failure signature?** `Destination Host Unreachable` reported *from the VM's own IP* (ARP for the gateway got no answer), followed by DNS failure and NetBird `dial context deadline exceeded`. Every upper-layer symptom was a downstream effect of having no layer-2 path to the gateway.

**How did I diagnose it?** Ping gateway, ping a public IP, resolve a name, then check the application (NetBird). The first failing step (gateway ping) located the fault at layer 2/3 boundary, so the NetBird error could be ignored until the network was fixed. The earlier incidents were located with `Get-NetIPAddress`, traceroute and `resolvectl status`.

**Root cause:** VM NIC VLAN tag didn't match the intended VLAN, so frames were tagged for the wrong network (or untagged onto the flat network) and never reached the right OPNsense VLAN interface. What *triggers* the tag to change (host restart, hardware edit, something else) is **not established** — it is the open question from three incidents.

**Fix:** Correct the tag in Proxmox → Network Device, then a full Stop → Start of the VM (not just a guest reboot).

---

## 4. What would I do differently next time?

- Check the Proxmox NIC (bridge + tag) **first** whenever a VM can't reach its gateway, before touching anything inside the guest.
- After any Proxmox host restart or hardware edit, audit all VM NICs in one pass rather than waiting for each VM to fail. On the host: `qm config <vmid> | grep net` for each VM.
- Monitor reachability so a drifted VM is noticed in minutes, not 12 days later (Uptime Kuma pinging each VM — already on the roadmap in Module 6).
- Record each VM's expected bridge/tag in the build log so "what was it before?" is never a guess.
- Give the critical VMs static IPs on their own tag and note it, since DC01's DHCP-vs-static regression was a direct consequence of landing on the wrong VLAN.

---

## 5. Real-world context

Named, public postmortem for a VLAN-tag error: **not found yet.** The search returned generic troubleshooting references, not a specific incident, so I am not claiming one.

**Link:** [My Trunk Won't Pass VLAN 20 — A 6-Step Debug Workflow](https://packetmentor.com/blog/trunk-not-passing-vlan/) (troubleshooting reference, not a postmortem)
**Connection:** Same failure category — a VLAN/trunk mismatch that looks like a dead host or broken application. Still worth finding a named incident on a future pass.

---

## 6. Explain it to a junior colleague

A VLAN tag is a label stuck on every network frame saying which virtual network it belongs to. In Proxmox, each VM's virtual network card carries one tag, and the firewall only answers frames labelled for the network it serves. If the tag is wrong or missing, the VM is effectively plugged into a different network: it can have a perfectly valid IP and still can't find its gateway. The tell is "Destination Host Unreachable" coming from the VM's own address. Check the tag on the VM's network device first, fix it, and fully stop and start the VM so the change takes hold.

---

## 7. Variation test (do this on a later pass, not required same-day)

**Variable changed:** (proposed, not yet done) Deliberately set a test VM's tag to a wrong VLAN, then to none, and record the exact symptoms for each (IP assigned? gateway ARP? which DHCP scope answers?).
**Result:** (pending)

---

## 8. Tags

`#VLAN-tagging` `#layer-2-reachability` `#ARP` `#proxmox-vlan-aware-bridge` `#layered-diagnosis` `#static-vs-DHCP-addressing` `#configuration-drift` `#monitoring-gaps`
