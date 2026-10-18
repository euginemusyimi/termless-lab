# Lab Task Debrief — NetBird remote access into the homelab (after WireGuard/OpenVPN were blocked)

## 0. Task identifier

**Date:** 2026-09-20 to 2026-10-12
**Task/lab name:** NetBird mesh VPN with an Ubuntu routing-peer VM into the 10.10.0.0/16 homelab
**Related project:** MSP Homelab Part 1 (VPN row), Part 6 Phase 8, Part 7 (Student-Facing IT / TEAM ia access layer)
**Time spent:** (fill in)

---

## 1. What was I trying to do?

Reach the homelab (VLANs 10/20/30/40) remotely without inbound port forwarding, because the ISP blocks inbound traffic (see the WireGuard debrief).

---

## 2. What did I actually do?

1. Chose self-hosting the NetBird control plane on an Oracle Cloud free-tier VM. Blocked twice: no free ARM (A1.Flex) capacity in Toronto, then the tenancy's region-subscription quota stopped me adding a second region. Pivoted to the hosted NetBird SaaS control plane (self-hosting parked).
2. Built `netbird-router`, an Ubuntu Server VM in Proxmox (vmbr1, VLAN tag 20), static IP `10.10.20.50/24` (see netplan debrief).
3. Installed the NetBird client, enrolled it with a setup key, confirmed `Management/Signal: Connected`.
4. Re-read NetBird's current docs and found **Network Routes are deprecated in favour of Networks**; rebuilt the config that way: groups (`homelab-routers`, `homelab-resources`, `remote-clients`), a Network `homelab` with a resource `10.10.0.0/16`, an access policy `remote-clients` to `homelab-resources`, and the router VM as routing peer.
5. Joined a domain VM and an iPhone; the phone on cellular reached the lab (user-confirmed).

```
sudo netbird up --setup-key <key>          # router VM enrolment
sudo netbird status -d                     # Management/Signal/Relays, peer status, Networks
cat /proc/sys/net/ipv4/ip_forward          # routing peer needs forwarding enabled
```

---

## 3. What broke — on purpose or not?

**Broke on purpose? (Y/N):** N

**What was the failure signature?** Several, in sequence: Oracle `Out of capacity for shape VM.Standard.A1.Flex`; tenancy region-subscription quota error; router VM `Destination Host Unreachable` from its own IP after 12 idle days (VLAN tag); my own instructions using the deprecated Routes model; dashboard layout changes (Setup Keys moved into Settings).

**How did I diagnose it?** Capacity and quota errors were provider limits, not config faults; I switched approach rather than fight them. For the router VM, gateway ping first, then the application. For the Routes/Networks mismatch, I read the current docs after a dashboard mismatch instead of trusting earlier instructions.

**Root cause:** Not one cause. The core lesson is that a design problem (ISP blocks inbound) was solved by changing the connection direction (outbound-initiated, relay fallback) rather than fixing the block. The detours were provider capacity, a recurring VLAN tag fault, and stale instructions on a fast-moving product.

**Fix:** Hosted control plane + routing peer + Networks/groups/policy.

---

## 4. What would I do differently next time?

- Check the vendor's current docs before building, especially for SaaS products that redesign dashboards and deprecate features.
- Have a no-quota fallback ready when depending on free-tier cloud capacity.
- Build the router VM's monitoring in first so a drifted VLAN tag is noticed immediately.
- Tighten the lab policy from `All` protocol to specific ports once the path is proven.

---

## 5. Real-world context

Named public incident: **not found yet.**

**Link:** [How Routing Peers Work – NetBird Docs](https://docs.netbird.io/manage/networks/how-routing-peers-work) (design reference, not a postmortem)
**Connection:** Real-world remote-access designs rely on the same pattern: routing peers inside the site, default-deny policies, and outbound-initiated connectivity so no inbound firewall holes are needed.

---

## 6. Explain it to a junior colleague

Normal VPNs need the office to accept incoming connections, which fails if the ISP blocks them. NetBird flips it: every device connects out to a coordination service, and the devices then set up direct tunnels (or a relay) with each other. A small Linux VM inside the network acts as the doorway into the whole network, and a policy says which groups of people may go through it. Nothing is allowed by default; you have to create the group, the resource, and the policy.

---

## 7. Variation test (do this on a later pass, not required same-day)

**Variable changed:** (proposed, not yet done) Remove the access policy and confirm the phone loses access while staying connected; then restore it scoped to specific ports only.
**Result:** (pending)

---

## 8. Tags

`#mesh-VPN` `#NAT-traversal` `#routing-peer` `#default-deny-policy` `#groups-and-policies` `#outbound-initiated-connectivity` `#vendor-docs-drift` `#cloud-capacity-quota` `#lazy-connection`
