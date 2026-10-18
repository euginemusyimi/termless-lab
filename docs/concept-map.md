# Lab Concept Map Index

> A living index, separate from your task-by-task debriefs. Update this every time you close out a task debrief — pull the tags from Section 8 and file the task under each concept below. Over weeks, this becomes your real study map: not "what did I do," but "what do I actually know, and how many angles have I seen it from."

---

## How to use this

1. Finish a task debrief (using the Lab Task Debrief Template).
2. Take the `#tags` from Section 8.
3. For each tag, add a row to that concept's table below (create a new concept section if it doesn't exist yet).
4. Every few tasks, skim the "Coverage check" at the bottom.

---

## Concept: Service Health vs. Liveness

**One-line definition (in your own words, update as understanding deepens):**
A service reporting "running" (process alive, valid PID) is not the same as a service actually performing its function correctly — liveness checks and functional checks answer different questions.

**Real-world roles/scenarios this shows up in:**
- Monitoring/alerting design (health checks that only ping a process vs. checks that verify a real transaction)
- Any port-sharing conflict between two services that both claim the same protocol/role
- Production incident response — "the dashboard says green but users report it's broken"

| Task / Lab | Tool used | What angle did this task add? | Broke on purpose? (Y/N + result) |
|---|---|---|---|
| OPNsense Kea DHCP port-67 conflict | `configctl`, `sockstat`, Kea logs | First real encounter: `configctl kea status` reported healthy while Kea was functionally dead for DHCP because dnsmasq held the port. Taught the specific diagnostic move of checking socket ownership (`sockstat`) instead of trusting service-manager status. | N — real post-reinstall misconfig, not injected |

**Related concepts:** `#socket-diagnostics` `#silent-failure` `#DHCP`

**Still fuzzy on:**
- [ ] Haven't yet tested the reverse ordering (dnsmasq starting after Kea) to see if the failure signature or which service "wins" changes — flagged as a variation test to run later

---

## Concept: DHCP Service Conflicts

**One-line definition:**
When multiple DHCP-capable services are enabled on the same host, only one can bind to UDP port 67 — the other keeps running as a process but cannot serve leases.

**Real-world roles/scenarios this shows up in:**
- Router/firewall appliance migrations (e.g. legacy ISC DHCP → Kea transitions, exactly what happened here)
- Any fresh OS/appliance install where multiple network services ship enabled-by-default

| Task / Lab | Tool used | What angle did this task add? | Broke on purpose? (Y/N + result) |
|---|---|---|---|
| OPNsense Kea DHCP port-67 conflict | Kea, dnsmasq, OPNsense WebUI | Learned that Kea's "Firewall rules" auto-checkbox only opens the DHCP protocol through the firewall — it does nothing to resolve a port-binding conflict with another local service. Two entirely different failure categories that look similar from the outside. | N |

**Related concepts:** `#service-health-vs-liveness` `#post-reinstall-config`

**Still fuzzy on:**
- [ ] Haven't tested this same conflict pattern on a non-BSD system (e.g. Linux `ss`/`netstat` equivalent) — flagged for a future Linux module (Module 6) task

---

## Concept: Socket-Level Diagnostics

**One-line definition:**
Checking which process actually owns a given network port/socket at the OS level, independent of what any service manager or dashboard claims.

**Real-world roles/scenarios this shows up in:**
- Debugging "service says running but isn't working" across any platform
- Security/incident response (confirming what's actually listening vs. what should be)

| Task / Lab | Tool used | What angle did this task add? | Broke on purpose? (Y/N + result) |
|---|---|---|---|
| OPNsense Kea DHCP port-67 conflict | `sockstat -4 -p 67` (BSD/OPNsense) | First hands-on use of socket-level ground truth as the tiebreaker when a service's own status output was misleading. | N |

**Related concepts:** `#service-health-vs-liveness` `#DHCP`

**Still fuzzy on:**
- [ ] Need the Linux-equivalent command(s) (`ss -ulnp`, `netstat -tulpn`) practiced hands-on, not just known conceptually — natural fit for Module 6 (Linux)

---

## Concept: External Reachability Testing

**One-line definition (in your own words, update as understanding deepens):**
Verifying whether a service is actually reachable from *outside* your own network, using a tool or vantage point independent of any equipment you control — the only way to distinguish "my config is wrong" from "something upstream is blocking me."

**Real-world roles/scenarios this shows up in:**
- Self-hosting anything (VPN, mail server, web server) on residential internet
- Diagnosing "works internally, fails externally" tickets — a very common MSP pattern
- Confirming NAT/firewall/ISP configuration changes actually took effect end-to-end, not just locally

| Task / Lab | Tool used | What angle did this task add? | Broke on purpose? (Y/N + result) |
|---|---|---|---|
| WireGuard remote access — ISP blocking inbound | canyouseeme.org | First real use of an external ground-truth tool. Learned the hard way this should be run *early* in a chain of local-config checks, not last — it would have cut hours of local troubleshooting by immediately pointing "upstream" rather than "local misconfiguration." | N |

**Related concepts:** `#NAT` `#ISP-blocking` `#silent-failure`

**Still fuzzy on:**
- [ ] Haven't yet tried other external test tools (e.g. `nc`/`nmap` from a remote VPS I control) to compare against a third-party site like canyouseeme.org

---

## Concept: NAT & Double-NAT Topologies

**One-line definition:**
Network Address Translation lets multiple internal devices share one public IP; when two NAT layers stack (ISP/home router NATing, then a second router/firewall NATing again behind it), inbound port-forwarding has to be configured correctly at every layer, and misconfiguration at any single layer breaks the whole chain.

**Real-world roles/scenarios this shows up in:**
- Homelabs where a dedicated firewall (OPNsense/pfSense) sits behind an ISP-provided router instead of replacing it
- Enterprise environments with multiple NAT boundaries (DMZ, cloud VPCs behind on-prem NAT, etc.)

| Task / Lab | Tool used | What angle did this task add? | Broke on purpose? (Y/N + result) |
|---|---|---|---|
| WireGuard remote access — ISP blocking inbound | OPNsense Interfaces Overview, TP-Link admin panel | Discovered OPNsense's WAN interface had a private IP (`192.168.0.37`) via DHCP from the home router — confirmed a real double-NAT topology in the lab, not just a textbook concept. | N |
| OPNsense Kea DHCP port-67 conflict | — | (cross-reference — same lab, different layer) | — |

**Related concepts:** `#port-forwarding` `#CGNAT` `#external-reachability-testing`

**Still fuzzy on:**
- [ ] Haven't tested what happens with 3+ stacked NAT layers (e.g. adding a cloud VPS relay in front, which would technically add a third hop) — natural follow-up once the VPS-relay workaround gets built

---

## Concept: ISP-Level Traffic Blocking / CGNAT

**One-line definition:**
Some ISPs block unsolicited inbound connections by policy (regardless of port), and separately, some assign shared "public-looking" IPs via Carrier-Grade NAT that aren't actually uniquely routable back to a single customer — both produce identical-looking symptoms (inbound traffic silently never arrives) but have different root causes and different fixes.

**Real-world roles/scenarios this shows up in:**
- Anyone self-hosting on residential/consumer-tier internet
- MSP client sites where "just port-forward it" doesn't work despite correct configuration
- Deciding between residential vs. business-tier ISP plans for a client with an inbound-service requirement

| Task / Lab | Tool used | What angle did this task add? | Broke on purpose? (Y/N + result) |
|---|---|---|---|
| WireGuard remote access — ISP blocking inbound | `curl -4`/`curl -6 ifconfig.me`, router's own WAN status page, canyouseeme.org, port-change test | Learned the specific two-step test to distinguish CGNAT (compare router's self-reported IP against an external site's view — if they match, it's not CGNAT) from general ISP inbound blocking (change the port and retest externally — if the new port also fails, it's not port-specific filtering, it's blanket inbound blocking). | N — real ISP constraint, confirmed the hard way |

**Related concepts:** `#NAT` `#double-NAT` `#external-reachability-testing` `#cloud-relay-pattern`

**Still fuzzy on:**
- [ ] Don't yet know whether this specific ISP would lift the restriction on request (e.g. business-tier plan, or a support call) — untested

---

## Concept: IPv4/IPv6 Dual-Stack Behavior

**One-line definition:**
A network connection can have both a valid public IPv4 address and a valid public IPv6 address simultaneously, and different tools/devices may default to reporting or using different ones — leading to confusing mismatches if not checked explicitly for both.

**Real-world roles/scenarios this shows up in:**
- Any modern ISP connection (most now provide dual-stack)
- VPN/remote-access configs where client and server must agree on which IP family to use
- General network troubleshooting — "why does this tool show a different IP than that one"

| Task / Lab | Tool used | What angle did this task add? | Broke on purpose? (Y/N + result) |
|---|---|---|---|
| WireGuard remote access — ISP blocking inbound | `curl ifconfig.me` vs `curl -4`/`curl -6`, iOS vs desktop WireGuard client comparison | First real encounter — iOS auto-picked an IPv6 endpoint while the desktop config used IPv4, for what was supposed to be the same tunnel. Learned to force explicit address-family checks (`-4`/`-6` flags) rather than trusting a bare `curl ifconfig.me` to reliably return the "right" one. | N |

**Related concepts:** `#NAT` `#external-reachability-testing`

**Still fuzzy on:**
- [ ] OPNsense's WAN interface never actually received a *public* IPv6 address via DHCPv6 (only link-local) despite the Windows machine on the same network getting one — root cause of that discrepancy not yet investigated

---

## Concept: Firewall Rule Field Semantics (Source vs. Destination)

**One-line definition:**
On an inbound WAN rule, "Source" means where the traffic is *coming from* (must be `any` for traffic from unpredictable internet clients), while "Destination" means what the traffic is *addressed to* (correctly the firewall's own WAN address) — confusing these two produces a rule that looks plausible but silently never matches real traffic.

**Real-world roles/scenarios this shows up in:**
- Any firewall/ACL configuration across any vendor (this exact Source/Destination confusion is a universal mistake, not OPNsense-specific)

| Task / Lab | Tool used | What angle did this task add? | Broke on purpose? (Y/N + result) |
|---|---|---|---|
| WireGuard remote access — ISP blocking inbound | OPNsense Firewall Rules | Caught two real mistakes in the same rule over the course of debugging: first an over-broad "any/any" rule, then later a genuinely wrong `Source: WAN address` (should have been `any`) that would have silently prevented all real client traffic from matching, independent of the ISP-blocking issue found later. | N — real mistakes, not injected |

**Related concepts:** `#firewall-rule-misconfiguration` (see Part 1 ACL work also)

**Still fuzzy on:**
- [ ] None currently — this one passed the "explain to a junior" test cleanly in the debrief

---



---

## Concept: VLAN Tagging & Layer-2 Reachability

**One-line definition:**
A VLAN tag labels each frame with the virtual network it belongs to; if a VM's NIC tag doesn't match the intended VLAN, the VM can hold a valid-looking IP yet never reach its gateway, because the layer-2 path (ARP) fails before routing begins.

**Real-world roles/scenarios this shows up in:**
- Any virtualised or switched environment with segmented networks (MSP client sites, data centers)
- "Host is up but unreachable" tickets after hypervisor or switch changes
- Network segmentation audits (are workloads actually on the VLAN the design says)

| Task / Lab | Tool used | What angle did this task add? | Broke on purpose? (Y/N + result) |
|---|---|---|---|
| VLAN tag mismatch on Proxmox VM NICs (DC01, client VM, netbird-router) | Proxmox Hardware → Network Device, `Get-NetIPAddress`, traceroute, `ping` gateway | Three recurrences taught the signature: `Destination Host Unreachable` coming from the VM's *own* IP means ARP to the gateway failed. Fix is at the hypervisor NIC, then a full Stop → Start. | N — recurring real fault; trigger still unknown |

**Related concepts:** `#ARP` `#proxmox-vlan-aware-bridge` `#configuration-drift` `#static-vs-DHCP-addressing`

**Still fuzzy on:**
- [ ] What exactly causes a VM's tag to change or be lost (host restart? hardware edit?) — not established across three incidents

---

## Concept: Layered Diagnosis Order (L2 → L3 → DNS → Application)

**One-line definition:**
Test from the bottom of the stack up — link/ARP, then gateway, then an external IP, then DNS, then the application — so the first failing step tells you which layer to fix and the application's own error can be ignored until the layers below it work.

**Real-world roles/scenarios this shows up in:**
- First-line support triage ("the app says it can't connect")
- Any outage where the visible error is an application timeout caused by a network fault

| Task / Lab | Tool used | What angle did this task add? | Broke on purpose? (Y/N + result) |
|---|---|---|---|
| VLAN tag mismatch on Proxmox VM NICs | `ping` gateway / `1.1.1.1` / hostname, `netbird status` | The NetBird `dial context deadline exceeded` error was a pure downstream symptom; gateway-ping failure located the fault in one step. | N |

**Related concepts:** `#VLAN-tagging` `#service-health-vs-liveness` `#external-reachability-testing`

**Still fuzzy on:**
- [ ] None flagged

---

## Concept: Network Config Renderers & Silent Config Failure

**One-line definition:**
On Ubuntu, netplan only translates YAML into settings for a renderer (systemd-networkd or NetworkManager); config can be accepted without error yet never take effect (wrong interface name, renderer not re-applying), so actual state must be checked, not assumed.

**Real-world roles/scenarios this shows up in:**
- Linux server provisioning and static addressing
- Cloud-image/automation work where config is "applied" but the host isn't configured as intended

| Task / Lab | Tool used | What angle did this task add? | Broke on purpose? (Y/N + result) |
|---|---|---|---|
| Static IP ignored on Ubuntu (netplan + NetworkManager) | `ip a`, `netplan get`, `nmcli connection show`, `nmcli connection up` | `netplan get` shows the renderer in control; a nonexistent interface name is ignored silently; the networkd warning was a red herring when NetworkManager was the renderer. | N |

**Related concepts:** `#service-health-vs-liveness` `#static-vs-DHCP-addressing` `#silent-failure`

**Still fuzzy on:**
- [ ] Why NetworkManager didn't re-apply the regenerated profile on its own

---

## Concept: Outbound-Initiated Connectivity & Mesh VPNs

**One-line definition:**
When inbound connections are blocked (ISP policy, CGNAT), have every endpoint connect *outbound* to a coordination service so peers can form tunnels directly (NAT traversal) or through a relay, instead of requiring an open inbound port.

**Real-world roles/scenarios this shows up in:**
- Remote access to sites on residential/consumer ISPs or behind CGNAT
- Zero-trust access products and site-to-site connectivity without firewall holes
- Contractor/offshore staff access to internal resources

| Task / Lab | Tool used | What angle did this task add? | Broke on purpose? (Y/N + result) |
|---|---|---|---|
| NetBird remote access into the homelab | NetBird (hosted control plane), `netbird-router` routing-peer VM, `netbird status -d` | The direction of connection initiation is the design lever: WireGuard/OpenVPN needed inbound; NetBird clients and the routing peer only dial out (STUN/TURN relays visible in status). Lazy connection means a peer shows Idle until traffic is sent. | N — worked around the ISP block; phone on cellular reached the lab (user-confirmed) |

**Related concepts:** `#NAT` `#ISP-level-blocking` `#external-reachability-testing` `#lazy-connection`

**Still fuzzy on:**
- [ ] Whether the phone test went direct peer-to-peer or via relay — not checked (`Connection type` in `netbird status -d`)

---

## Concept: Default-Deny Access Policies (Groups → Resources)

**One-line definition:**
Access is denied until a policy explicitly allows a source group to reach a destination resource group; the groups, the resource and the policy are separate objects and all three must line up.

**Real-world roles/scenarios this shows up in:**
- Zero-trust / least-privilege access design
- Giving contractors or offshore staff access to only the systems they need
- Firewall rule design generally

| Task / Lab | Tool used | What angle did this task add? | Broke on purpose? (Y/N + result) |
|---|---|---|---|
| NetBird remote access into the homelab | NetBird Networks (groups, resource `10.10.0.0/16`, policy) | A peer can be visible and connected yet pass no traffic until group membership and a source-to-destination policy match. The deprecated Routes model allowed access by default; Networks is default-deny. Lab policy was intentionally broad (all protocols) to be tightened later. | N |

**Related concepts:** `#firewall-rule-field-semantics` `#least-privilege` `#groups-and-policies`

**Still fuzzy on:**
- [ ] Tighten the policy to specific ports and confirm only those work

---

## Coverage check (review every ~5–10 tasks)

- **Concepts touched only once:** Service Health vs. Liveness (now also related to the netplan debrief), DHCP Service Conflicts, Socket-Level Diagnostics, External Reachability Testing, NAT & Double-NAT Topologies, ISP-Level Traffic Blocking/CGNAT, IPv4/IPv6 Dual-Stack Behavior, Firewall Rule Field Semantics, VLAN Tagging & Layer-2 Reachability (one debrief, but covers 3 incidents), Layered Diagnosis Order, Network Config Renderers & Silent Config Failure, Outbound-Initiated Connectivity & Mesh VPNs, Default-Deny Access Policies — 13 concepts, 5 debriefs done. Expected at this stage; look for second angles as more Part 1 incidents and Part 2 modules get debriefed.
- **Concepts touched 3+ times from different angles:** none yet.
- **Concepts I still can't explain without notes:** none flagged — both debriefs so far passed the "explain to a junior" test cleanly.
- **Real-world case studies collected so far:** 2 (Kea/dnsmasq port-conflict pattern; residential ISP inbound-blocking/CGNAT pattern) — both generic community-knowledge references so far, still worth finding a specific named incident/CVE/postmortem for at least one of these on a future pass.

---

## Cross-concept connections

> The real payoff of this index: seeing how concepts combine in real systems (e.g. VLANs + ACLs + routing = segmentation strategy). Log these when you notice them.

| Concepts combined | Where this shows up in real infrastructure | Task(s) that revealed this |
|---|---|---|
| NAT/Double-NAT + External Reachability Testing + ISP-Level Blocking | Diagnosing "can't reach my self-hosted service from outside" tickets — the standard troubleshooting order is: confirm local config → confirm NAT/port-forward at every hop → test from a genuinely external vantage point → only then suspect the ISP. Real MSP/homelab pattern, not homelab-specific. | WireGuard remote access — ISP blocking inbound debrief |
| Outbound-Initiated Connectivity + ISP-Level Blocking + NAT | When inbound is blocked (ISP/CGNAT), change the direction of the connection rather than fight the block. The WireGuard diagnosis proved the limitation; NetBird's outbound-only design worked around it. | WireGuard ISP-blocking debrief + NetBird debrief |
| VLAN Tagging & Layer-2 Reachability + Layered Diagnosis Order | "App can't connect" tickets: test gateway → external IP → DNS → app, and the first failing step names the layer. The NetBird timeout was really a VLAN-tag fault. | VLAN tag mismatch debrief |
| Network Config Renderers + Service Health vs. Liveness | "Applied without error" is not "in effect": `netplan apply` succeeded while the interface kept its DHCP lease. Same meta-lesson as Kea/dnsmasq — check real state. | netplan/NetworkManager debrief |
| Service Health vs. Liveness + Socket-Level Diagnostics | Both incidents (Kea/dnsmasq and WireGuard) shared the same meta-lesson: "the tool/dashboard says it's fine" is not sufficient evidence — always verify with a lower-level, more direct check (socket ownership in one case, external reachability in the other) before trusting a status indicator. | dnsmasq/Kea port conflict debrief + WireGuard ISP-blocking debrief |

---

## Why this system works: professional practices it mirrors

> This index and the debrief template aren't arbitrary — they compress habits that experienced IT professionals build over years into a per-task routine. Reference this section when motivation dips or when deciding what to prioritize.

1. **Lab journals / "build logs" — almost universal among senior engineers**
Experienced sysadmins and network engineers keep a running log of every non-trivial thing they configure, especially in home labs. Not for show — because in six months they've forgotten why they set a particular MTU or NAT rule, and the log saves hours of re-discovery. This is exactly the debrief template, just done habitually over years instead of per-lab.

2. **"Break it to understand it" — a documented troubleshooting discipline**
Senior network/sysadmins deliberately misconfigure things in lab/test environments before they ever touch production, specifically to know what the failure signature looks like ahead of time. When something breaks for real at 2am, they're pattern-matching against failures they've already seen on purpose. This is the "break it on purpose" step, just formalized as a career-long habit, not a one-off lab trick.

3. **Teaching/documenting for others (rubber duck at scale)**
Writing internal runbooks, documentation, or answering junior engineers' questions is consistently reported by senior IT staff as one of the things that most solidified their own understanding — because it forces the "explain it like I'm onboarding someone" step (Debrief Section 6). Many people who blog, write internal wikis, or mentor say they learned the topic better by writing about it than by doing it the first time.

4. **Certification labs + real incident/CVE study side by side**
People preparing for things like CCNA, CompTIA Security+, or AWS certs who do well long-term don't just lab the objectives — they pair each lab topic with a real breach report, postmortem, or CVE writeup (exactly the "real-world context" step). Security folks especially do this constantly: lab a misconfiguration, then go read the actual incident report where that misconfiguration caused a real breach.

5. **Spaced repetition / redo without notes**
One of the most evidence-backed learning techniques in general (not just IT) — Anki decks for cert prep, or simply redoing a lab cold a few times in different ways, are common among people passing hard certs and retaining it. The "variation test" step mirrors this directly.

6. **Cross-referencing concepts instead of tools in isolation**
Senior engineers think in terms of "segmentation strategy" or "identity and access model," not "I ran this VLAN command." When troubleshooting, they reason from the concept down to the specific config, not the other way around. This is exactly what this concept-map index trains — building a mental model organized by systems/concepts rather than by individual commands memorized in sequence.

7. **Deliberate practice with increasing difficulty, not repetition of the same task**
A well-known finding from skill-acquisition research (not IT-specific, but heavily applied in IT training) is that repeating the same easy task doesn't build skill — you need tasks slightly above your current ability, with feedback. In IT this looks like: once a lab is "done," professionals change a variable (different topology, injected fault, different constraint) rather than just repeating the same steps. The "change one variable and see what breaks" step matches this directly.
