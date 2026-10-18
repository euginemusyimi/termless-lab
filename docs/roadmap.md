# Master Lab & Career-Readiness Roadmap

This is the single living document tracking all homelab/project work, mapped against target job postings and the full Infra Admin Lab Plan. Update this whenever new labwork, practices, or job descriptions come in via a **"project alignments"** message.

---

## How "project alignments" works

When you type `project alignments` followed by new labwork, practices, job descriptions, or projects:
1. I read what's pasted/described
2. I cross-reference it against everything already in this doc — flagging overlaps so we don't duplicate work already done
3. I update this roadmap: new modules/add-ons get added, new job postings get their own alignment table, statuses get updated
4. We move into implementing the next concrete piece of labwork, same working style as the MSP build

---

## Part 0 — Documentation & Retention System (applies to everything below)

This isn't a module or a project — it's the practice layer that should wrap around *every* task in Parts 1 and 2 going forward. Two linked artifacts:

1. **Lab Task Debrief** — filled out per task/lab session immediately after finishing (captures what was done, what broke, real-world context, "explain it to a junior" pass, tags)
2. **Lab Concept Map Index** (this document you provided) — a separate, cumulative index built *from* the tags in each debrief, organized by concept rather than by task, so understanding compounds across unrelated labs instead of staying siloed per-project

**Why this matters for the roadmap specifically:** your MSP homelab work (Part 1) has already generated multiple concepts worth indexing — VLAN segmentation, DHCP service conflicts, DNS hierarchy/forwarding, firewall rule evaluation order, Proxmox VLAN-tag/hardware gotchas, AD/GPO application — but none of it has been run through this system yet. Retroactively debriefing the completed Part 1 work would immediately populate several concept sections and prove the system out before applying it prospectively to new modules.

### Practices this formalizes (from the source doc, condensed)
- Build logs — same habit senior engineers keep, just structured
- "Break it to understand it" — deliberate lab fault injection before it happens for real
- Teaching/explaining as a comprehension test, not just an output
- Pairing lab topics with real breach reports/postmortems/CVEs
- Spaced repetition — redoing labs cold, without notes
- Reasoning from concept → config, not command → command
- Deliberate practice — varying one constraint per repetition rather than repeating identically

### Status
| Item | Status |
|---|---|
| Debrief template | ✅ Built — `lab-task-debrief-template.md` |
| Concept Map Index | ✅ Built and populated — `lab-concept-map-index.md` (13 concepts logged across 5 debriefs; see the index for the list) |
| Retroactive debrief of Part 1 MSP homelab work | 🟡 In progress — 5 done (dnsmasq/Kea port conflict; WireGuard ISP-blocking incident; **VLAN tag mismatch recurrence** covering DC01, client VM and netbird-router; **netplan/NetworkManager static-IP failure**; **NetBird remote access**) — 13 concepts logged. Next debrief: OPNsense GUI exposure / block-rule segmentation test (once the rule is verified). Remaining strong candidates: physical NO-CARRIER cable fault, OPNsense kernel-panic reinstall itself, Oracle Cloud free-tier capacity/region-quota blocker |
| As-built record (current state of the lab) | 🟡 Drafted 2026-10-17 — `homelab-as-built.md`; known facts filled, blanks marked `(fill in)`. Update with every change; pairs with the git `homelab` repo from Part 6 and, later, the BookStack/Hudu SOP library from Part 9 |
| Ongoing use for Part 2 modules going forward | 🔲 Not started |

**Suggested next step:** debrief the VLAN-tag/DC-DHCP cascade next — it's a multi-layer incident (Proxmox hardware mistag → traceroute diagnosis → DC losing static IP back to DHCP) that will populate several new concepts and start producing the first Cross-Concept Connections entries once it's logged alongside the DHCP conflict debrief.

---

## Part 1 — Hands-On Core: MSP Homelab (Proxmox + OPNsense)

This is your primary hands-on project and satisfies large chunks of Modules 1, 2, and 4 below through real (not simulated) infrastructure.

| Item | Status |
|---|---|
| Proxmox VE host, vmbr0/vmbr1, VLAN-aware trunk | ✅ Done |
| 4x VLANs (MGMT/SERVERS/WORKSTATIONS/DMZ), trunking, inter-VLAN routing | ✅ Done |
| NAT, Kea DHCP (survived real dnsmasq/port-67 conflict), firewall ACLs | ✅ Done |
| Windows Server 2025 DC (`DC01`), AD DS + DNS, promoted forest `TermlessSolution.local` | ✅ Done |
| OU structure, test users/groups (PowerShell + GUI both practiced) | ✅ Done |
| GPOs: password/lockout policy (Default Domain Policy), GPP drive mapping | ✅ Done |
| Domain-joined Windows client, verified via `gpresult`, cross-VLAN (client on VLAN 30 → DC on VLAN 20) | ✅ Done — survived a genuine multi-layer outage (Proxmox VLAN mistag, physical NO-CARRIER cable fault, DC losing static IP back to DHCP) |
| VPN — NetBird remote access (WireGuard on OPNsense was blocked) | ✅ **NetBird working end-to-end (2026-10-12, user-confirmed)** — hosted SaaS control plane (self-hosting on Oracle free tier parked after capacity + region-quota blockers) with an Ubuntu routing-peer VM `netbird-router` (10.10.20.50, VLAN 20) using the current **Networks** model (Routes are deprecated). Debrief: `debrief-netbird-remote-access.md`. Original WireGuard status follows for history. 🟡 WireGuard: Config complete, functionally blocked — root-caused to ISP-level inbound blocking (confirmed via port-change test + canyouseeme.org; not fixable locally). Documented as a full debrief with real diagnostic chain. **Two fixes now identified:** (1) Part 6 Phase 8 (site-to-site to a cloud VPC) — outbound-initiated from OPNsense, sidesteps the ISP block entirely, but requires standing up and paying for/maintaining a cloud VPS; (2) **NetBird (identified 2026-09-20, project alignment)** — free, open-source, self-hostable WireGuard-based mesh overlay using ICE/WebRTC for P2P NAT traversal with TURN relay fallback. **Needs zero inbound port forwarding**, so it sidesteps the same ISP block with far less infrastructure than a full cloud VPC. Recommended as the next thing to actually try — cheaper/faster proof than Phase 8, and doubles as the access layer for the new TEAM ia/Ellucian mock lab in Part 7. Phase 8 remains worth doing separately as its own hybrid-cloud-networking skill, just no longer the only path forward. |
| VPN — OpenVPN on OPNsense | 🔲 Not started — next up (expect the same ISP-blocking wall for direct home access; build for skill demonstration regardless, per earlier decision) |
| Backup/restore — Proxmox Backup Server + Veeam Community Edition | 🔲 Not started — Veeam has a free Community Edition plus free training videos; worth running both PBS (Proxmox-native) and Veeam (industry-standard name, appears in job posting preferred lists) side by side. Set up a real backup job, then actually break something and restore it — the restore, not just the backup job, is the part worth demonstrating |
| Monitoring/centralized logging | 🔲 Not started — **concrete stack now defined via Part 6 Phase 4:** Prometheus + Grafana + node_exporter for metrics, Loki or ELK/OpenSearch + Promtail for logs. **Fast first win added:** Uptime Kuma (Module 6) — lightweight, quick to stand up, good initial monitoring win before committing to the heavier full stack |
| Azure/Entra hybrid join, automation script, EDR trial | 🔲 Not started — EDR trial now has a concrete option: Wazuh (Part 6 Phase 9), in addition to the earlier Defender for Endpoint idea |
| Segmentation hardening — block VLAN 20 from OPNsense admin GUI (80/443) | 🟡 **In progress (2026-10-13/15).** Nmap from `netbird-router` showed GUI ports open on 10.10.20.1 and 12 AD ports on DC01. A SERVERS block rule was added, but the verification scan still showed 80/443 open, so the rule is not yet effective (check rule order, interface tab, Apply changes, log view). Needs a before/after scan and a debrief. See Part 8 Nmap |
| Branch office, voice/guest/IoT VLANs, FreePBX, QoS, config-restore SOP | 🔲 Not started — **see Part 9 (MSP Business Track)** for the full scenario list and sequencing |

**Real incident narrative captured so far** (strong Lab Evidence / interview material): OPNsense kernel-panic reinstall → dnsmasq/Kea port-67 conflict diagnosed via `sockstat` → physical NO-CARRIER cable fault mid-project → Proxmox VLAN-tag client misplacement (traceroute-diagnosed) → DC01 same VLAN-tag issue → DC losing static IP to DHCP after a hardware change (root-caused via `Get-NetIPAddress` showing `PrefixOrigin: Dhcp`). Every layer of the OSI stack got debugged at least once.

---

## Part 2 — Infra Admin Lab Plan (10-Module Program)

Cross-cutting practice standard applies to every module below: ticketing/documentation habit, troubleshooting methodology (identify → reproduce → isolate → check logs → hypothesize → test → confirm → document), customer-communication framing, identity-as-onboarding-story mindset, and the reactive-support → proactive-sysadmin mindset shift (pattern recognition, scaling logic, change-management questions, capacity thinking).

| Module | Focus | Status | Overlap with MSP homelab |
|---|---|---|---|
| **1** | Azure/Virtualization (AZ-104, VMware add-on, storage) | 🔲 Not started | Proxmox virtualization work transfers conceptually; Azure/VMware still net-new. **New:** also spin up Hyper-V or VMware ESXi specifically (a second physical box, or nested inside Proxmox) — job postings frequently name Hyper-V/VMware by brand, not just "virtualization" generically. AZ-104 pairs directly with Module 3 (M365/Entra/Intune) — worth sequencing them back-to-back |
| **1a** | *Add-on:* Cloud Networking Fundamentals (AWS + Azure) — see detail below | 🔲 Not started | Subnetting/CIDR concepts transfer directly from VLAN work; VPC/VNet, cloud-native routing, security groups still net-new |
| **2** | Windows Server Hybrid Admin & Identity (AD, OU, GPO, Entra Connect, MFA/SSO, onboarding/offboarding) | 🟡 Partial | **AD DS/DNS, OU structure, GPOs, domain join = done.** Entra Connect/MFA/SSO/dynamic groups/onboarding-offboarding docs = not yet |
| **3** | Microsoft 365 Administration | 🔲 Not started | None yet. **New:** explicitly includes Entra ID and Intune — pairs with AZ-104 (Module 1) since almost every posting touches Microsoft cloud in some form; Intune also overlaps with Module 7's device-management angle |
| **4** | CCNA/CCNP (routing/switching/NOC) | 🟡 Partial | **VLANs, trunking, inter-VLAN routing, ACLs, DHCP/DNS, TCP/IP fundamentals = done** (via OPNsense/Kea, not Cisco IOS — real, working implementation, just a different vendor). **Genuine gap requiring real Cisco IOS behavior:** OSPF, STP, BGP, HSRP/VRRP — OPNsense/Proxmox can't simulate actual Cisco CLI/protocols, so this needs **GNS3 or EVE-NG** specifically (network simulators that run real or emulated Cisco IOS images). Wireshark packet analysis and Juniper/SD-WAN add-ons also still open |
| **4a** | *Add-on:* Network Automation & SDN — see detail below | 🔲 Not started | None yet — net-new topic area |
| **4b** | *Add-on:* SD-WAN (Cisco Viptela architecture) — see detail below | 🔲 Not started | None yet — net-new topic area, but conceptually related to Module 4a's controller-based/SDN theme |
| **4c** | *Add-on:* GNS3/EVE-NG Cisco Simulation Labs — see detail below | 🔲 Not started | **Fills Module 4's real gap** — the specific tool for practicing OSPF/STP/HSRP-VRRP with actual Cisco IOS behavior, which OPNsense can't provide |
| **4d** | *Add-on:* Data Center Networking Vendor Landscape — see detail below | 🔲 Not started | Reference/awareness only, not hands-on — knowing which company makes what and what engineers do with each category |
| **5** | Palo Alto/Fortinet Security | 🔲 Not started | OPNsense firewall/NAT/ACL work is conceptually adjacent but not Palo Alto-specific. **PCNSA confirmed as the explicit near-term cert target** for this module (matches Posting B's highest-priority gap) |
| **6** | Linux Administration | 🔲 Not started | None yet. **Concrete project plan (expanded):** install Debian or Ubuntu on a home server; host real self-hosted services to force genuine CLI usage — Pi-hole (DNS-level ad blocking, good pairing/contrast with the OPNsense/Kea DNS work in Part 1), Nextcloud (file sync/self-hosted cloud), Ansible control node (see Module 9 overlap below), RHEL install + Docker containers for distro/containerization exposure. **New additions, all free:** Docker + **Portainer** (web UI for managing Docker — much easier to demo/screenshot than raw CLI alone); **Uptime Kuma** (lightweight self-hosted uptime/status monitoring — a fast, low-effort complement to Part 6 Phase 4's heavier Prometheus/Grafana stack, good first monitoring win before tackling the full stack); **Vaultwarden** (self-hosted Bitwarden-compatible password manager — genuinely useful secrets-management practice, and a good real service to put behind Nginx Proxy Manager below); **Nginx Proxy Manager** (reverse proxy with a GUI — natural fit for the still-unused **DMZ VLAN (40)** from Part 1, since a reverse proxy is exactly the kind of service that belongs in a DMZ); **Wazuh** (SIEM/EDR — same tool already flagged in Part 6 Phase 9, this confirms it as the right choice, no change needed there) |
| **7** | Desktop Support/Imaging/Mac/SCCM | 🔲 Not started | None yet |
| **8** | Bash & Python Scripting | 🔲 Not started | None yet. **Refined split:** PowerShell for the Windows/AD side (already has real material to script against from Part 1 — user/OU creation, GPO checks), bash/Python for the Linux side once Module 6's home server exists — natural to build scripts *for* the Module 6 services (e.g. a Python script hitting Pi-hole's API, a bash backup script for Nextcloud) rather than writing scripts in isolation |
| **9** | Ansible (Configuration Management) | 🔲 Not started | Shares config-mgmt tooling (Ansible/Puppet/Chef) with Module 4a above — build once, counts for both. Module 6's home server plan explicitly includes standing up Ansible there — sequence Module 6 first so Ansible has a real target to manage rather than a throwaway VM. **Concrete playbook targets now defined (Part 6 Phase 5):** users, packages, hardening |
| **9a** | *Add-on:* CI/CD / GitOps — see Part 6 Phase 6 | 🔲 Not started | **Genuine gap, newly surfaced** — Gitea/GitLab CE + a runner that lints/applies Ansible + Terraform on commit. No prior overlap; sequence after Module 9/10 exist, since there's nothing to lint/apply before those are built |
| **10** | Terraform (Infrastructure as Code) | 🔲 Not started | None yet. **Concrete project now defined (Part 6 Phase 5):** Terraform or OpenTofu with the Proxmox provider — provisioning your own real infrastructure as code, rather than a disconnected tutorial project |
| **10a** | *Add-on:* Containers & Orchestration (Docker Compose, K3s) — see Part 6 Phase 7 | 🔲 Not started | **Genuine gap, newly surfaced** — Docker itself overlaps with Module 6's home-server plan, but K3s/Kubernetes (deployments, services, ingress, Helm) is entirely net-new and high-priority given how often Kubernetes fluency is named as a current hiring signal |
| **Capstone** | Full hybrid environment tying all 10 together | 🔲 Not started | — |

### Module 1a detail — Cloud Networking Fundamentals (AWS + Azure)

Vendor-neutral cloud networking foundation. Directly serves the "Azure or AWS networking" preferred skill (Posting A) and the network-segmentation/Zero Trust angle in Posting B. Several concepts here transfer directly from Part 1 VLAN/subnet work — this is "the same mental model, cloud-native vocabulary."

| Topic | Status | Notes |
|---|---|---|
| IP & Subnetting — CIDR mastery, public vs. private IPs, subnet calculations | 🔲 Not started | **Strong overlap** with Part 1's VLAN subnetting (`10.10.x.0/24` design work) — mostly a vocabulary/context translation, not new conceptual ground |
| VPC (AWS) / VNet (Azure) — isolated cloud network, where resources live | 🔲 Not started | Cloud-native equivalent of the vmbr1 trunk + VLAN design in Part 1 — good candidate to explicitly diagram side-by-side with the homelab topology for the Lab Evidence writeup |
| Subnets & Routes — public vs. private subnets, route tables, Internet Gateway, NAT Gateway | 🔲 Not started | Route table logic is conceptually close to OPNsense's routing; Internet Gateway/NAT Gateway are the cloud-native equivalents of vmbr0 + OPNsense's outbound NAT |
| Security Groups — AWS Security Groups, Azure NSGs, allow/block by port & IP | 🔲 Not started | Directly parallels the OPNsense pass-rule work already done (source/destination/port ACLs) — same logic, different console |
| Load Balancing & DNS — ALB, NLB, Route 53 / Azure DNS, high availability | 🔲 Not started | Net-new — no direct Part 1 equivalent yet |
| Hybrid Connectivity — VPN Gateway, Direct Connect / ExpressRoute, on-prem↔cloud | 🔲 Not started | **Direct continuity with Part 1's WireGuard/OpenVPN work** — same remote-access-into-a-segmented-network concept, cloud-vendor version. Natural module to tackle once Phase 3 (VPN) wraps up in Part 1 |

**Sequencing note:** this pairs unusually well with what's already in progress — IP/Subnetting and Security Groups are near-direct translations of VLAN/ACL work already proven in Part 1, and Hybrid Connectivity is a natural next step right after the WireGuard/OpenVPN work finishes. Worth tackling this module earlier than its numbering suggests, since the marginal effort is low given existing homelab experience. **Cert path note:** AZ-104 must come before AZ-305 (hard prerequisite) — treat AZ-104 as the gate for this whole module before AZ-305-specific solution-architecture study begins.

### Module 4a detail — Network Automation & SDN

CCNA 200-301-aligned "Automation and Programmability" domain. Sits alongside Module 4 (routing/switching) and shares tooling with Module 9 (Ansible) — building the config-management piece once counts toward both.

| Topic | Status | Notes |
|---|---|---|
| Why network automation matters (manual CLI vs. automated config at scale, drift/consistency problems it solves) | 🔲 Not started | Conceptual foundation — pairs well with the troubleshooting-methodology practice already used in the MSP homelab (manual config today, automated tomorrow) |
| Controller-based networking / SDN (control plane vs. data plane separation, centralized policy) | 🔲 Not started | OPNsense/Proxmox setup so far has been traditional box-by-box config — SDN is a conceptual shift worth contrasting against that experience |
| Northbound & Southbound APIs (controller-to-app vs. controller-to-device) | 🔲 Not started | Foundational vocabulary before touching DNA Center or any REST tooling |
| REST APIs & JSON (structure, verbs, auth, parsing responses) | 🔲 Not started | Direct prerequisite for Module 8 (Python scripting) and Module 10 (Terraform) — good candidate to tackle early since it unlocks later modules |
| Configuration management tools — Ansible, Puppet, Chef (concepts + comparison) | 🔲 Not started | **Overlaps directly with Module 9** — the conceptual/comparison piece belongs here, hands-on playbook-writing happens in Module 9 |
| Cisco DNA Center (controller platform, assurance, automation workflows) | 🔲 Not started | Vendor-specific — most realistic as reading/conceptual study unless a DNA Center sandbox/trial becomes available; not replicable in the Proxmox/OPNsense lab as-is |

**Sequencing note:** REST APIs & JSON is the highest-leverage single topic here — it's a direct prerequisite for both Module 8 (Python/Bash scripting) and Module 10 (Terraform/IaC), so tackling it before those two modules pays off twice.

### Module 4b detail — SD-WAN (Cisco Viptela architecture)

Vendor-specific (Cisco SD-WAN/Viptela terminology) but the underlying concepts — centralized control plane, policy-driven routing, overlay tunnels — are the same pattern seen in Module 4a's SDN discussion and in Module 1a's cloud VPN/hybrid connectivity topic. Treat this as the "how a real enterprise WAN vendor implements SDN principles" case study.

| Topic | Status | Notes |
|---|---|---|
| SD-WAN Architecture (overlay/underlay, fabric design) | 🔲 Not started | Conceptual entry point — good to pair with Module 4a's control-plane/data-plane split discussion |
| Controllers & Control Plane (vManage, vSmart, vBond roles) | 🔲 Not started | Direct extension of Module 4a's "controller-based networking" concept, now with concrete vendor role names |
| OMP (Overlay Management Protocol) | 🔲 Not started | Viptela's control-plane routing protocol — conceptually parallel to how OSPF/BGP distribute routes in traditional networking (Module 4), but centralized rather than distributed |
| TLOC (Transport Locator) | 🔲 Not started | SD-WAN's addressing/reachability abstraction over underlying WAN transports |
| Templates (device/feature templates for centralized config push) | 🔲 Not started | Conceptually close to Module 4a/9's configuration-management theme — centralized template push vs. Ansible playbook push are solving the same problem from different vendors |
| Centralized & Local Policies (traffic steering, app-aware routing) | 🔲 Not started | Policy-based routing — parallels firewall/ACL rule-writing already practiced in Part 1, just applied to path selection instead of pass/block |
| Routing & VPNs (service VPNs, segmentation within the fabric) | 🔲 Not started | **Conceptually close to Part 1's VLAN segmentation work** — SD-WAN "service VPNs" are functionally a WAN-scale version of the same isolation principle |
| Troubleshooting | 🔲 Not started | Best tackled last, once the architecture pieces above are understood — same "concept before command" principle the Debrief system (Part 0) is built around |

**Sequencing note:** this is largely vendor-specific reading/conceptual study unless a Cisco SD-WAN lab/sandbox becomes available (not replicable in the Proxmox/OPNsense lab as-is — similar constraint to DNA Center in Module 4a). Highest-value approach: work through it conceptually, then explicitly map each concept back to something already built in Part 1 or Module 4a (the Routing & VPNs topic above is the clearest example of this) rather than treating it as entirely isolated new material.

### Module 4c detail — GNS3/EVE-NG Cisco Simulation Labs

Fills the specific gap Module 4 flagged: real Cisco IOS behavior (OSPF, STP, HSRP/VRRP) that OPNsense/Proxmox genuinely cannot simulate, since OPNsense is a different vendor/OS entirely, not Cisco IOS underneath. GNS3 and EVE-NG both run real or emulated Cisco device images inside VMs — this is the standard, widely-used path for CCNA/CCNP-level hands-on practice without buying physical Cisco hardware.

| Topic | Status | Notes |
|---|---|---|
| GNS3 or EVE-NG installation (choose one — both run inside Proxmox as a VM) | 🔲 Not started | EVE-NG tends to be preferred for larger/more complex topologies and has a strong community image library; GNS3 has a gentler learning curve and tighter Cisco-official-image integration. Either is a legitimate choice — pick one rather than splitting effort across both |
| OSPF — build a multi-router topology, verify convergence, break a link and observe reconvergence | 🔲 Not started | Direct fill for Module 4's flagged OSPF gap. Natural first debrief candidate for this module — pairs well with Part 0's "break it on purpose" practice |
| STP — build a switched topology with redundant links, observe blocking-port behavior, then intentionally misconfigure priorities and watch root bridge election change | 🔲 Not started | STP misconfiguration (wrong root bridge, missing BPDU guard) is a classic real-world outage cause — strong candidate for a debrief with real-world postmortem pairing (Part 0 practice #4) |
| HSRP/VRRP — build a redundant gateway pair, fail one over, confirm seamless client failover | 🔲 Not started | Directly answers Posting B's "HA datacenter architecture" requirement |
| Wireshark packet capture inside GNS3/EVE-NG topologies | 🔲 Not started | GNS3/EVE-NG can capture directly on virtual links — good pairing with Module 4's flagged Wireshark gap, using traffic generated by the labs above rather than needing a separate capture setup |

**Sequencing note:** OSPF is the natural first lab here — it's the most commonly tested CCNA topic, and its "break a link, watch it reconverge" pattern is the cleanest first debrief for this module. Do this before STP/HSRP, since OSPF failure/recovery behavior is easier to reason about than STP's port-state transitions.

**Cross-reference to Part 0:** every lab in this module is an ideal candidate for the debrief system — GNS3/EVE-NG labs are explicitly designed to be broken and rebuilt repeatedly (the "variation test" and "redo without notes" practices), which is exactly what a simulator environment is *for*, unlike physical/production infrastructure where deliberate breakage carries real risk.

### Module 4d detail — Data Center Networking Vendor Landscape

Industry-awareness reference, not a hands-on lab — knowing *what a company makes* and *what an engineer actually does with that gear day-to-day* is genuinely interview-relevant (resumes/job postings name vendors constantly), even without owning the hardware. Grouped by function rather than listed individually, since that's how this knowledge gets used in practice.

| Category | Companies | What network engineers commonly do with this gear |
|---|---|---|
| **Core switching/routing (the CCNA/CCNP world)** | Cisco, Arista, Juniper, HPE/Aruba, Extreme Networks, H3C, Ruijie, Huawei, Nokia, ZTE | Configure and troubleshoot VLANs, spanning-tree, routing protocols (OSPF/BGP), spine-leaf data-center fabrics, ACLs — **this is exactly what Module 4/4c already targets**, these are simply the vendors whose CLI/OS that knowledge applies to. Arista in particular is heavily associated with large-scale data-center leaf-spine designs |
| **Compute/server hardware** | Dell Technologies, HPE, Lenovo, Inspur, Penguin Solutions, QCT | Rack/cable physical servers, manage out-of-band controllers (iDRAC on Dell, iLO on HPE), firmware/BIOS updates, hardware lifecycle — more sysadmin/data-center-ops territory than pure networking, but engineers touch this when a server's network connectivity is the actual problem |
| **Silicon, chipsets & AI-cluster networking** | Nvidia, Broadcom, Mellanox Technologies, Marvell Technology | Lower-level than switch CLI config — NIC/DPU (data processing unit) configuration, RDMA/InfiniBand fabric design for AI/ML training clusters. Nvidia's acquisition of Mellanox is why "AI networking" and "traditional networking" increasingly overlap — worth knowing as a trend even without hands-on access, since AI-cluster network design is a fast-growing specialty |
| **Optics, transceivers & white-box/disaggregated networking** | FS.com, Accton Technology, Edgecore Networks, Lumentum, AOI Electronics, Diamond Communications, QCT | Select and install SFP/QSFP transceivers, fiber patch/cabling standards, and increasingly deploy "white-box" switches running an open/disaggregated network OS (e.g. SONiC) instead of a vendor's proprietary OS — a real, growing alternative to buying Cisco/Arista/Juniper hardware outright, common at hyperscale/large-scale shops |

**Practical takeaway for interviews:** you don't need hands-on time with all 26 — the useful skill is recognizing which functional bucket a named vendor falls into when it comes up in a job posting or conversation, and being able to say what an engineer actually *does* with that category of gear, rather than just recognizing the logo.

---

## Part 6 — DevOps/SRE/Cloud Engineering Extension (9-Phase Program)

A sequential, git-documented build program targeting Sysadmin/Infra, DevOps/SRE, Cloud Engineering, and Network Engineering roles. Builds on the *same physical Proxmox foundation* as Part 1, but several phases deliberately take a **different technical path** (Linux/FreeIPA identity vs. Windows AD, containers/K8s vs. traditional VM services) — genuinely complementary rather than duplicate, since it demonstrates range across both Windows-shop MSP environments and Linux-native DevOps/SRE environments.

| Phase | Focus | Status | Overlap / relationship to existing work |
|---|---|---|---|
| **1** | Network Segmentation — OPNsense/pfSense VM, VLANs (mgmt/servers/DMZ/lab), inter-VLAN routing, firewall rules | ✅ **Already done** | Directly satisfied by Part 1's OPNsense/VLAN build — same core skill, no rework needed |
| **2** | Centralized Identity — FreeIPA or Samba AD DC, join Linux VMs via SSSD, centralize users/groups/sudo | 🔲 Not started | **Genuinely net-new**, not a duplicate of Part 1's Windows AD work — FreeIPA/LDAP is the Linux-native identity paradigm and a distinct, valuable skill for DevOps/SRE postings specifically. Worth doing *in addition to*, not instead of, the Windows AD work already done |
| **3** | DNS & DHCP — BIND9/dnsmasq internal zone, DHCP via OPNsense/isc-dhcp-server | 🟡 Partial | DHCP delivery pattern already proven (Kea/OPNsense from Part 1); BIND9 zone specifically not built. Naturally pairs with Phase 2 — FreeIPA typically wants to manage its own DNS zone, so sequence these two together |
| **4** | Monitoring & Logging — Prometheus + Grafana + node_exporter; Loki or ELK/OpenSearch + Promtail | 🔲 Not started | **This is the concrete tool stack for Part 1's existing "Monitoring/centralized logging" item** — previously vague, now specific. Update made below. **Update (2026-10-15): Part 9 adds LibreNMS or Zabbix for SNMP/network-device monitoring (switches, radios, UPS, WAN). Pick whichever fits; note the reason in a debrief.** |
| **5** | Automation & IaC — Ansible playbooks (users/packages/hardening); Terraform/OpenTofu with Proxmox provider | 🔲 Not started | **This is the concrete project for Module 9 (Ansible) and Module 10 (Terraform)** in Part 2 — Terraform-against-Proxmox specifically is a strong, realistic IaC project since it targets infrastructure you already own. Updated below. |
| **6** | CI/CD — Gitea or self-hosted GitLab CE; runner that lints/applies Ansible + Terraform on commit | 🔲 Not started | **Real gap** — no existing module in Part 2's 10-module plan covers CI/CD/GitOps at all. This is a genuinely new topic area worth its own tracked line (see below) |
| **7** | Containers & Orchestration — Docker Compose migration; K3s cluster across 2–3 VMs (deployments, services, ingress, Helm) | 🔲 Not started | Docker itself already appears in Module 6's home-server plan (previous batch), but **K3s/Kubernetes is a significant net-new topic** not covered anywhere in the current plan — flagged as high-priority given Kubernetes fluency is named here as a top 2026 hiring signal |
| **8** | Hybrid Cloud & Networking — small VPC on AWS/Azure/GCP free tier; site-to-site WireGuard/IPsec, OPNsense ↔ cloud VPC | 🔲 Not started | **Directly rescues the blocked WireGuard work from Part 1.** A cloud VPS/VPC has a real, unrestricted public IP — site-to-site *from* OPNsense *to* the cloud is an outbound-initiated connection, which sidesteps the ISP inbound-blocking root cause found in the WireGuard debrief entirely. This is effectively the "cloud relay" workaround already proposed, formalized as its own phase. **Update (2026-09-20, project alignment): NetBird identified as a faster, cheaper alternative fix for the same ISP-blocking problem** — free, self-hosted, WireGuard-based mesh VPN with P2P NAT traversal (ICE/WebRTC) + TURN relay fallback, requiring no inbound port forwarding at all. Recommend trying NetBird first (low cost, fast to stand up, immediately unblocks Part 1's VPN row and the new TEAM ia/Ellucian mock lab in Part 7); keep this Phase 8 cloud-VPC build on the roadmap regardless, since site-to-site cloud networking is its own distinct, valuable skill for the DevOps/SRE track — not purely a workaround. |
| **9** | Security Tooling — Wazuh (SIEM/EDR) fleet-wide; Suricata IDS on OPNsense; periodic OpenVAS/Nessus-essentials scans | 🔲 Not started | Wazuh covers Part 1's monitoring gap from a security angle and the EDR/MDR required skill (Posting A); Suricata runs natively as an OPNsense plugin, so it's a low-friction add to infrastructure already built. **Update (2026-09-30, project alignment): this phase now has a dedicated, sequenced skill-building track — see new Part 8 (SOC Analyst Skill-Building)** — Wazuh here is literally the same tool as Part 8's "build your own SIEM" weekend project, just tracked from the DevOps/SRE angle here vs. the SOC-analyst angle there. Do the work once, it satisfies both. |

### New gaps this batch surfaced (not covered anywhere else in this roadmap)
- **CI/CD / GitOps (Phase 6)** — no existing module touches this at all; worth adding as its own tracked module, likely "Module 9a" alongside Ansible
- **Kubernetes/K3s (Phase 7)** — same situation; genuinely new, high-priority given the "top 2026 hiring signal" framing

### Operating principles (apply across all 9 phases)
- **Document everything in a git repo** (`homelab`): playbooks, Terraform configs, network diagrams, a README per phase. This runs *alongside* Part 0's debrief/concept-map system rather than replacing it — the git repo is the artifact record, the debrief system is the learning/retention record. Both should reference each other (e.g. a debrief citing the specific commit/playbook it was written about).
- **Keep resource usage lean**: LXC for lightweight services, full VMs only where kernel isolation is genuinely required (firewall, FreeIPA, K3s nodes) — a real constraint on single-PC hardware worth respecting rather than defaulting to full VMs everywhere.
- **Do not skip Phases 1–3** — later phases assume working VLANs/DNS/identity. Since Phase 1 is already done and Phase 3 is partially done, Phase 2 (FreeIPA) is the actual next gate before Phases 4+ become sensible to start.

### Certification alignment (this program's track, cross-check against Part 4)
- Sysadmin: RHCSA or **LPIC-1** (LPIC-1 not previously in Part 4's cert table — added below as an alternative to RHCSA)
- Network Engineering: CCNA (already tracked)
- Cloud: associate-level cert for chosen provider (already tracked generically via Module 1/1a)
- DevOps/Kubernetes: **CKA**, after Phase 7 (new — added to Part 4's cert table below)

---

## Part 3 — Job Posting Alignment (tracked per posting)

### Posting A — MSP L3 Systems Administrator
*(Original target posting — see earlier roadmap for full requirement breakdown)*

Best-served by: Module 2 (partial, in progress), Module 4 (partial), Part 1 homelab directly. VPN/backup/monitoring items in Part 1 still open and are explicitly named in this posting's Lab Evidence wishlist.

### Posting B — Senior Network & Security Administrator (FDM4 — ERP/eCommerce, multi-datacenter)

| Posting requirement | Covered by |
|---|---|
| Enterprise LAN/WAN, VLANs, HA datacenter architecture | Module 4 (VLANs, HSRP/VRRP, 3-tier topology) — VLANs/trunking already proven in Part 1 |
| Layer 2/3, trunking, spanning-tree, BGP/OSPF | Module 4 — trunking done; STP/BGP/OSPF not yet |
| Palo Alto firewalls, Panorama, NAT/Zero Trust segmentation | Module 5 — not started; **highest-priority gap for this posting** |
| Network monitoring, capacity planning, packet analysis | Module 4 (Wireshark) + Module 5 (traffic/threat logs) — not yet |
| 50+ site-to-site VPNs | Module 5 VPN section |
| GlobalProtect | Module 5 |
| General IT support beyond networking | Modules 1, 2, 3, 6, 7 broadly |
| Preferred certs: PCNSA, PCNSE, CCNP, Security+, CISSP | See cert table below |

**Takeaway:** this posting is network/security-senior-leaning — if actively pursuing it, Module 5 (Palo Alto) deserves priority ahead of Linux/scripting/IaC, since it's this employer's stated day-one expectation.

*(New postings get added here as their own subsection when pasted via "project alignments.")*

---

## Part 4 — Certification Targets

| Module | Certs |
|---|---|
| 1 (Azure/Virtualization) | AZ-104; VMware VCP-DCV if pursuing the VMware add-on. **AZ-104 → AZ-305 path confirmed:** AZ-305 requires AZ-104 as a hard prerequisite, so this is a fixed sequence, not a choice. **Free/low-cost lab path:** Azure Free Account ($200 credit, first 30 days — use for intensive lab weeks); Microsoft Learn sandboxes (genuinely free, unlimited, built into the official learning paths — the sustainable option once the $200 credit runs out); Azure for Students ($100 credit, no card required, if eligible). Exam fees themselves (~$165 each) aren't avoidable, but all prep/lab time can be free. |
| 2 (Hybrid Identity) | Windows Server Hybrid Administrator Associate, SC-300 (Identity and Access Administrator) — modern replacement for retired MCSA |
| 3 (Microsoft 365) | MS-102 (M365 Administrator Expert), or Modern Desktop Administrator track |
| 4 (Routing/Switching) | CCNA → CCNP Enterprise; Juniper JNCIA if posting calls for it |
| 5 (Palo Alto/Fortinet) | PCNSA → PCNSE; Fortinet NSE 1-3 (free/fast) if Fortinet-leaning posting |
| 6 (Linux) | RHCSA or LPIC-1 |
| 7 (Desktop Support) | CompTIA A+, Apple Certified Support Professional (ACSP) |
| 9a/10a (CI/CD, Kubernetes) | CKA (Certified Kubernetes Administrator), pursued after Part 6 Phase 7 |
| General baseline | Security+ |
| Service management | ITIL Foundation |
| Senior/leadership | CISSP (once experienced) |

---

## Part 5 — Extras Worth Doing (resume gold, not certs)

- Public homelab writeup (blog/GitHub README) — the ongoing Lab Evidence documentation already builds toward this
- Cross-platform PowerShell script to round out the scripting story
- A sample ticket queue (10–15 realistic tickets, logged and resolved with priority/resolution notes) — directly answers "show me you understand the ticketing process"
- **ITIL exposure** — not a hands-on lab item, but a service-management framework worth basic fluency in (incident/problem/change management vocabulary). ITIL Foundation is a short, affordable cert if pursuing formally; even without the cert, being able to frame homelab incidents using ITIL terms (e.g. "this was a Problem, not just an Incident, since it recurred and needed root-cause analysis" — genuinely applicable to the dnsmasq/Kea and WireGuard debriefs already written) is worth doing for interview language alone

---

## Part 7 — Verified Professional Experience → Free Practice Mapping

Source: a real-world skills inventory from actual IT work (higher-ed environment). This differs from the rest of the roadmap in one important way — these are skills *already demonstrated on the job*, not aspirational modules. The purpose here is mapping each to a free way to keep them sharp / prove them independently (for interviews, portfolio, or in case job access disappears), and being honest about the couple of things that genuinely can't be replicated outside that specific job.

| Skill Area | Topics Mastered | Free practice path (2026-current) | Cross-reference |
|---|---|---|---|
| **Microsoft 365 Administration** | Shared mailboxes, Exchange Online, Entra ID, Conditional Access, MFA, licensing | **Microsoft 365 Developer Program E5 sandbox** — free, 25 user licenses, 90-day renewable, includes Entra ID P1/P2 (so Conditional Access/MFA are testable). **Caveat confirmed via current search:** eligibility has tightened in 2026 — some users report needing a linked Visual Studio subscription to qualify; if the self-service sandbox isn't offered, fall back to Microsoft Learn's free scoped sandboxes for individual exercises | Ties to Module 3 and Module 1a |
| **Active Directory** | User management, account troubleshooting, GPO, ADUC, PowerShell AD commands | **Already fully covered** — this is exactly Part 1's DC01/AD build (real OUs, users, GPOs, both GUI and PowerShell practiced) | Part 1 — no new work needed, genuine overlap |
| **Network Troubleshooting** | tracert, Test-NetConnection, port testing, DNS, DHCP, Wi-Fi investigation | **Already fully covered, and thoroughly** — the WireGuard/OpenVPN debrief sagas used exactly this toolkit (tracert, Test-NetConnection, external port testing via canyouseeme.org, DNS/DHCP root-causing) against real, non-trivial problems | Part 1 — arguably your strongest proven area already |
| **Endpoint Security** | Antivirus management, malware response, Intune, MDM, BYOD | **Intune is bundled into the same M365 E5 dev sandbox above** — no separate signup needed. Malware response: Windows Defender is free/built-in for baseline exposure; Microsoft Defender for Endpoint also offers a trial | Ties to Part 6 Phase 9 (Wazuh) as the open-source EDR alternative if the trial lapses |
| **Teams & VoIP** | Teams Phone, call queues, voicemail, auto attendants, Ellucian integration | **Partially free, with an honest limit:** the M365 E5 dev sandbox includes Teams Phone System, so call queues/auto attendants/cloud voicemail are configurable and testable for internal (Teams-to-Teams) calls at no cost. **Real PSTN calling (actual inbound/outbound phone numbers) always costs money** — Calling Plans, Direct Routing (SBC + SIP trunk), and Operator Connect are all paid, confirmed via current search, no free path exists. **Ellucian is a proprietary higher-ed ERP/SIS with no public sandbox** — this piece genuinely can't be replicated outside that job. **Alternative for hands-on VoIP/PBX concepts specifically:** 3CX free tier (up to 10 users, self-hostable) gives real call-routing/PBX practice independent of Teams entirely. **Update (2026-10-15, project alignment): FreePBX (Sangoma) is now the primary PBX practice target — see Part 9 scenarios 10-12; 3CX stays optional for comparison** |
| **PDF & Document Management** | Foxit signatures, document protection, signature workflows | Foxit's free Reader tier covers basic viewing/annotation; for the **signature workflow** concept specifically, DocuSign's free tier (3 free envelopes) or Adobe Sign's trial demonstrate the same e-signature pattern. Document protection (password/permissions) is practiceable free directly in LibreOffice or Word within the M365 dev sandbox |  |
| **Backup & DR** | 3-2-1-1-0 rule, backup types, RTO, RPO, ransomware response | **Already covered** — Proxmox Backup Server + Veeam Community Edition (Part 1) | Part 1 backup row |
| **Windows Administration** | Windows 11 deployment, drivers, Bluetooth, print spooler, registry | **Windows 11 Enterprise evaluation ISO** — free 90-day eval from Microsoft, deployable in Proxmox alongside existing VMs. Print spooler/registry troubleshooting can be practiced directly on VMs already built (DC01, domain client) |  |
| **PowerShell Automation** | AD management scripts, M365 scripts, bulk operations, remote execution | AD scripting already has a real target (Part 1's DC01). M365 scripting needs the **Microsoft Graph PowerShell SDK** (free) pointed at the same M365 dev sandbox tenant above | Ties to Module 8 |
| **Cybersecurity** | Phishing defense, incident response, forensics, AADSTS errors | Phishing defense: **Attack Simulation Training**, bundled free into the same M365 E5 dev sandbox (part of Defender for Office 365). **AADSTS error codes are genuinely practiceable for free** — deliberately misconfigure Conditional Access or an app registration in the dev tenant and observe the real error codes generated; strong "break it on purpose" candidate for Part 0. Forensics: Eric Zimmerman's free forensic tools, or Event Viewer analysis already practiced in Part 1 |  |
| **Cloud & Azure** | Azure fundamentals, Defender for Cloud, cost management, hybrid | **Already covered** — Azure Free Account + Microsoft Learn sandboxes (Module 1). Defender for Cloud's foundational CSPM tier is free within the Azure Free Account | Module 1, Module 1a |
| **Student-Facing IT** | Ellucian, HTTP 500, application portals, international student support | **Ellucian Colleague itself remains non-replicable** — no public sandbox, proprietary higher-ed SIS. **Concrete alternative now identified (project alignment 2026-09-20): TEAM ia** — a real higher-ed eForms/workflow/eSignature platform that integrates directly with Ellucian Banner/Colleague, Anthology, Salesforce, and Workday. Practice plan: build a mock version of that integration pattern in the homelab — a simple self-hosted forms/workflow app (or a Nginx Proxy Manager–fronted app in Module 6's stack) standing in for TEAMia, feeding a mock "SIS" record store standing in for Colleague — then practice the same troubleshooting motions real TEAMia/Colleague integrations require: auth handoff failures, webhook/API errors, HTTP 500s on form submission, and DB record mismatches between the two systems. Access the whole mock environment remotely via **NetBird** instead of raw WireGuard (see Cross-reference) — this also keeps the HTTP 500 / web-app troubleshooting angle already identified here | **NetBird has a much bigger implication than just this row** — see Part 1 (WireGuard row) and Part 6 Phase 8 below |
| **IT Procurement** | License management, asset tracking, vendor management | **Snipe-IT** — free, open-source, self-hostable IT asset management tool. Genuinely good concrete project, and a natural addition to Module 6's home-server stack | Module 6 |
| **Career Development** | Certifications, documentation, communication skills | Not a technical lab item — this is exactly what Part 0 (debrief/concept-map system) and Part 5 (extras) already exist to build | Part 0, Part 5 |

**Honest summary of what can't be freely replicated:** Ellucian (proprietary SIS) and real PSTN phone calling (always paid, by every provider checked). Everything else on this list has a genuine, current, free path — either already built in this roadmap, or newly identified here.

---

## Part 8 — SOC Analyst Skill-Building Track (added 2026-09-30, project alignment)

Source: a "4 free tools to start looking like a SOC analyst" weekend-project framework. Cross-referenced against the rest of this roadmap below — two of the four tools were already present elsewhere (Wireshark, Wazuh); Splunk and Nmap are genuinely new additions, as is the explicit Kali-vs-Windows attack/defend lab pattern.

| Tool | What it builds | Status / cross-reference | Weekend project |
|---|---|---|---|
| **Wireshark** | Network traffic analysis + packet inspection | **Already tracked** — Part 2 Module 4a detail flags this exact gap, paired with GNS3/EVE-NG captures (line ~152). No new work needed, just do it | Open a sample capture (from Wireshark's own sample-capture site), filter for HTTP traffic, look for unencrypted passwords or odd DNS queries |
| **Splunk** | Log analysis + threat detection + dashboards | 🔲 **Genuinely new** — not covered anywhere else in this roadmap. Splunk Free (500MB/day ingestion) is a real, current free tier | Install Splunk on a VM (or the existing client VM), forward Windows Event Logs from DC01 or the domain client, run searches against real log data already being generated by Part 1's AD environment |
| **Wazuh** | SIEM setup + alerting + rule creation | **Already tracked twice** — Part 6 Phase 9 (DevOps/SRE security tooling) and Part 7's Endpoint Security row (EDR alternative). This weekend-project framing is the same tool, just approached from the "build a SOC alert and watch it fire" angle rather than the "deploy fleet-wide" angle. **One build satisfies all three references** — deploy Wazuh once, then do the SOC-style exercise (connect a Windows agent, trigger a test alert for a failed login) as the first thing you do with it | Set up Wazuh on Ubuntu, connect the Windows domain client as an agent, trigger a test alert for a failed login |
| **Nmap** | Network reconnaissance + vulnerability assessment | 🟡 **Started 2026-10-13** — first scans run against VLAN 20 from `netbird-router` (found OPNsense GUI exposed, DC01 AD ports, 3 hosts via ARP). Re-scan after the block rule is fixed, then from VLAN 30 for a segmentation comparison. **Genuinely new** — not explicitly named anywhere else in this roadmap, though it's a natural companion to the existing OpenVAS/Nessus-essentials line in Part 6 Phase 9 (Nmap finds what's there; OpenVAS checks it for vulnerabilities) | Run Nmap against the homelab's own VLANs (10.10.0.0/16) to map what's actually listening — a genuinely useful sanity check on the real environment already built, not just a toy target |

### Attack/defend lab pattern (Kali + Windows 11 in VirtualBox)
**Net-new pattern, not yet in this roadmap.** The source material suggests VirtualBox specifically, but **Proxmox (already the hypervisor for everything else in this roadmap) does this better** — no reason to introduce a second virtualization platform for two more VMs when the existing one has capacity and you already know it well. Recommended adaptation:
- Spin up a **Kali Linux VM** on its own isolated VLAN (or a new VLAN 50 — "RED TEAM" — kept deliberately unable to reach MGMT/20 or the real client devices, only a dedicated target)
- Point Kali's Nmap/vuln-scanning work at a **dedicated target VM** (reuse the existing Windows 11 domain-joined client, or better: clone a throwaway copy of it so nothing gets broken) rather than the production DC01
- This is the same "isolated attacker/defender" concept as the broader lab, just formalized with a named attacker box — genuinely good, low-cost addition

### Honest sequencing note
Wazuh is the one item here with real infrastructure dependencies (a VM, an agent install) — do it as part of Part 6 Phase 9 rather than twice. Wireshark needs zero new setup — it's just overdue. Splunk and Nmap are both quick, standalone, one-sitting wins with no dependency on anything else in this roadmap — good candidates to knock out whenever there's a short window between the longer VPN/networking work currently in progress.

---

## Part 9 — MSP Business Track: Network & Voice (added 2026-10-15, project alignment)

Source: *Home Lab to MSP: Network & Voice Business Plan* (5 phases). It reuses the Proxmox lab as a **training and testing workshop only**, never for client production, and adds three things the roadmap lacked: a **voice/PBX** stack, a **two-site network** with physical gear, and the **business setup** (tools, insurance, contracts, compliance, selling).

**How this fits the rest of the roadmap.** The job-application track (Parts 1-8) and this business track share one deliverable: **written SOPs from rehearsed scenarios**. An SOP you can show a hiring manager is the same SOP you would hand a client. Do the lab work once, write it up once, use it for both. Where the plan overlaps existing work it is cross-referenced below, not duplicated.

**Ground rules carried over from the plan:** the lab never hosts client data or phones (residential internet, no SLA, single points of failure, insurance and PIPEDA exposure); every infrastructure cost must be backed by recurring revenue; write service descriptions in your own words under your own brand; snapshot before every change; never port-forward the lab.

### 9.1 Phase 1 lab build vs what already exists

| Plan item | Roadmap status | Notes |
|---|---|---|
| vmbr0 (home network as the "internet") and vmbr1 (fake client office) | ✅ Done (Part 1) | vmbr1 is VLAN-aware with 4 VLANs, richer than the plan's single flat bridge |
| **vmbr2 plus a second firewall VM** (branch office) | 🔲 New | Needed for site-to-site VPN (scenario 9). Both bridges stay inside the lab, so the ISP inbound block is irrelevant |
| OPNsense/pfSense firewall, DHCP, VLANs, VPN, QoS | 🟡 Mostly done | Firewall, VLANs, Kea DHCP, rules done. QoS/traffic shaper and a branch firewall not yet |
| Windows Server + AD, DNS, GPO, file shares | ✅ Done (DC01) | File shares and GPP drive mapping done; share permissions not yet written up |
| 2× Windows 11 domain-joined PCs | 🟡 One of two | `j2iq606` is built. Second client is also the clone target for Part 8's attack/defend lab |
| Ubuntu VM: UniFi controller, test RMM, docs, LibreNMS/Zabbix | 🔲 New | Overlaps Module 6 home-server plan and Part 6 Phase 4 (see 9.3) |
| **Sangoma FreePBX** | 🔲 New | First voice item in the roadmap. Replaces 3CX as the main PBX practice target in Part 7's Teams & VoIP row (3CX stays an optional comparison) |
| Proxmox Backup Server | 🔲 Not started (Part 1 backup row) | Same item, not a second one. The plan adds a separate disk and **timed** restores |
| Microsoft 365 Business Premium trial | 🔲 Not started | Realistic SMB tenant (Entra, Intune, Exchange, Defender, conditional access). Complements Part 7's E5 developer sandbox, so use both. Check current trial terms before starting the clock, since trials are time-limited |
| Remote access without port forwarding | ✅ NetBird working (Part 1) | Plan suggests Tailscale or WireGuard. NetBird is the same class of tool and is already done |
| Snapshot before every change | 🟡 Habit | Make it a rule. It also fits Part 0's debrief template |

### 9.2 Scenarios to rehearse and write up as SOPs

| # | Scenario | Status | Cross-reference / what's missing |
|---|---|---|---|
| 1 | Onboard an employee (account, licence, Intune laptop, MFA, shared drives) | 🔲 | Module 2 (onboarding docs) + Module 3. Needs the M365 tenant. AD half is doable now on DC01 |
| 2 | Offboard (block sign-in, convert mailbox, transfer OneDrive, wipe device) | 🔲 | Same tenant. Pair with #1 so the SOPs mirror each other |
| 3 | Restore a deleted file and a whole VM, **timed** | 🔲 | Part 1 backup row. The timings give you real RTO numbers to quote |
| 4 | Monthly patch cycle (pilot PC, then roll out) | 🔲 | Module 7. Run via RMM or Intune once chosen |
| 5 | Compromised-account response (reset, revoke sessions, review sign-in logs) | 🔲 | Part 7 Cybersecurity row (AADSTS practice) + Part 8 Splunk/Wazuh log work |
| 6 | **Replace a firewall from an exported config** | 🔲 | **Best first scenario**: OPNsense config export, then restore to a fresh VM. No new infrastructure, and it protects the firewall rule work now in progress |
| 7 | New office network: VLANs for data, voice, guest, cameras/IoT with rules between them | 🟡 | Existing VLANs 10/20/30/40 and inter-VLAN rules done. Voice, guest and IoT VLANs not built. **VLAN 50 is already reserved for Part 8's RED TEAM**, so use 60 VOICE, 70 GUEST, 80 IoT/CAMERAS |
| 8 | Guest Wi-Fi: client isolation, bandwidth limit, captive portal | 🔲 | OPNsense captive portal can be tested without hardware. Wi-Fi isolation needs an AP |
| 9 | Site-to-site VPN between two firewalls, plus staff remote access | 🟡 | Remote access ✅ NetBird. Site-to-site needs vmbr2 + branch firewall. Distinct from Part 6 Phase 8, which is lab-to-cloud |
| 10 | Voice QoS: DSCP EF tagging, prioritise, test calls on a saturated link | 🔲 | Needs FreePBX + a softphone or desk phone + OPNsense traffic shaper |
| 11 | FreePBX build: extensions, ring groups, time conditions, IVR, voicemail-to-email, trunk, DID routing | 🔲 | Net-new. Real trunk calls cost money (pay-as-you-go SIP account) |
| 12 | PBX hardening: firewall module, intrusion detection, no SIP open to the world, strong secrets, international calling blocked, toll-fraud alerts | 🔲 | **Prove it with Nmap** (Part 8), which you are already practising on the OPNsense GUI exposure |
| 13 | Dual-WAN failover with LTE backup | 🔲 | Simulate with a second WAN NIC and a gateway group. Real LTE needs hardware |
| 14 | Power planning: PoE budget, UPS runtime, NUT on Proxmox | 🔲 | Hardware-dependent |
| 15 | PTP link: align two radios, record signal and throughput | 🔲 | Hardware-dependent. Defer until a client or trigger exists |

### 9.3 Phase 2: tool stack (trial in the lab first, as few vendors as possible)

| Need | Plan's options | Roadmap link |
|---|---|---|
| RMM | NinjaOne, N-able, Datto, Syncro; Tactical RMM (free) | Tactical RMM is the free trial path. Host it on a cloud VPS only if it ever manages real clients |
| PSA / ticketing + billing | Syncro, HaloPSA, ConnectWise, Autotask | Part 5's "10-15 sample tickets" can be logged in the trial PSA, so one effort covers both |
| Documentation | BookStack (free), Hudu, IT Glue | Hosts the SOPs from 9.2. Keep it alongside the git `homelab` repo and Part 0 debriefs |
| Client credentials | Bitwarden / Keeper business plans | Vaultwarden (Module 6) is fine for practice, but not for client production |
| Network management | UniFi, Cambium cnMaestro, firewall config backups | Config-backup piece is scenario 6 |
| Network monitoring | RMM SNMP, **LibreNMS or Zabbix** (free) | Adds SNMP coverage that Part 6 Phase 4 (Prometheus/Grafana/Loki) and Uptime Kuma don't. Pick one and note why in a debrief |
| Asset tracking | (not in plan) | Snipe-IT from Part 7 fits here too |
| Bookkeeping, M365 via CSP distributor, backup, security, SIP reseller, hardware distributors | Pax8, Veeam/Acronis/Datto SaaS, Defender/Huntress, Ubiquiti/Cambium/Netgate/Sangoma | Business-side only. Wazuh (Part 6 Phase 9 / Part 8) is the lab stand-in for the security-tooling concept |

### 9.4 Hardware shopping list (plan's list, ordered by what unblocks scenarios)

| Item | Unblocks | Suggested timing |
|---|---|---|
| UPS with USB/network interface (NUT on Proxmox) | #14 and protects the existing lab | Early. Useful even for the job-search lab |
| PoE switch + 1-2 UniFi APs | #8 (isolation), UniFi controller practice | When the Ubuntu VM with UniFi is ready |
| 1-2 IP desk phones (Sangoma/Yealink/Poly) | #10, #11 realism | After FreePBX works with a softphone first |
| Pay-as-you-go SIP trunk (e.g. VoIP.ms) | #11 real calls | After PBX hardening (#12) is done, never before |
| Entry-level PTP radios | #15 | Defer until a client site needs them |

### 9.5 Phase 3: business setup and compliance checklist

Confirm everything below with an accountant, lawyer and insurer. Two items from the plan need attention:

- **HST:** the plan only mentions the four-consecutive-quarters test. Per [TaxTips.ca](https://taxtips.ca/gst/howtoregister.htm), registration is also required if you pass $30,000 in a **single calendar quarter**, with a short deadline to register. Confirm the exact timing with CRA or your accountant.
- **CRTC:** the plan says "check whether you must register" if you resell SIP trunks. A [VoIP.ms reseller article (Jan 2026)](https://voip.ms/blog/voip-reseller-obligations/) says resellers must register with the CRTC before offering service, provide 9-1-1, register with the CCTS, and publish an accessibility plan. That is a vendor blog, not the regulator, so confirm on crtc.gc.ca before selling any trunk or DID.

| Item | Status |
|---|---|
| Register the business (Ontario Business Registry; sole proprietor or incorporate) | 🔲 |
| Decide on early HST registration (see note above) | 🔲 |
| Insurance: E&O, cyber, general liability covering on-site work | 🔲 Needed before touching any client system |
| Lawyer-reviewed MSA + scope-of-work templates (toll fraud, 9-1-1 limits, RMA, ISP vs your duties, privacy clause) | 🔲 |
| PIPEDA accountability, privacy clause in contracts | 🔲 |
| 9-1-1 for VoIP: correct civic address per DID, document it | 🔲 |
| CRTC reseller registration check | 🔲 |
| Working at Heights training (rooftop/tower work) | 🔲 Only if PTP work is offered |
| Separate business bank account + bookkeeping | 🔲 |

### 9.6 Phases 4-5: selling and growth (reference)

- **Door-openers:** IT assessment, phone bill audit, Wi-Fi/network health check, PBX security audit. Build templates for each. The **PBX security audit reuses scenario 12 and Part 8 Nmap work**.
- **Projects:** network design/install, PTP/PTMP links, phone system install, managed-IT onboarding. **Recurring:** managed IT (Essentials/Standard/Premium), managed network, managed phone, resold SIP trunks. Set prices only after checking local competitors, and price for profit.
- **Turnkey phone package:** one PBX build, one or two phone models, one trunk provider, under your own brand.
- **Solo limits:** business-hours support only, plus a backup-coverage partner.
- **Invest-on-trigger table** (hosted PBX or client apps move to a Canadian-region VPS/colo, spare stock at several sites, part-time tech past ~40 hrs/week, compliance tooling for regulated clients): kept as written in the plan. None of these are triggered yet.

### 9.7 Sequencing against the existing roadmap

The plan's "month 1" assumes the lab is built from scratch. Most of it exists, so the real remaining Phase 1 work is: **config-restore (#6) → branch firewall + site-to-site (#9) → extra VLANs (#7) → FreePBX (#11, #12, #10) → M365 tenant scenarios (#1, #2, #5) → timed restores (#3)**. The plan's dates are a starting guess. Use the Suggested Pacing table as the real schedule, and treat the business checklist (9.5) as a parallel admin track that costs money only once you decide to launch.

---

## Suggested Pacing (from source plan — adjust as MSP homelab work continues in parallel)

| Weeks | Focus |
|---|---|
| 1–3 | Module 1 |
| 4–6 | Module 2 (partial credit already banked from Part 1) |
| 7–8 | Module 3 |
| 9–12 | Module 4 (partial credit already banked from Part 1) |
| 13–15 | Module 5 |
| 16–17 | Module 6 |
| 18–19 | Module 7 |
| 20–21 | Module 8 |
| 22–23 | Module 9 |
| 24–25 | Module 10 |
| 26 | Capstone |
| 27 | Cert exams |

---

*Last updated (2026-10-15): merged project alignment — added Part 9 (MSP Business Track: Network & Voice) from the Home Lab to MSP business plan; FreePBX, branch firewall/site-to-site, voice/guest/IoT VLANs, LibreNMS/Zabbix and business setup are new, everything else cross-referenced. Also logged Nmap/OPNsense GUI hardening as in progress and refreshed the Part 0 debrief counts. Previous update (2026-09-30): merged project alignment — added new Part 8 (SOC Analyst Skill-Building Track: Wireshark, Splunk, Wazuh, Nmap, plus a Kali/Windows attack-defend lab pattern adapted to Proxmox instead of VirtualBox). Wireshark and Wazuh were already tracked elsewhere (Part 2 Module 4a, Part 6 Phase 9, Part 7 Endpoint Security) and cross-referenced rather than duplicated; Splunk and Nmap are genuinely new additions. Part 6 Phase 9 updated to point at Part 8. Previous update (2026-09-20): TEAM ia + NetBird merge (Part 7 Student-Facing IT, Part 1 WireGuard row, Part 6 Phase 8). Update via "project alignments" going forward.*
