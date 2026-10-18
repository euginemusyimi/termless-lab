# Homelab As-Built Record (TermlessSolution)

**Purpose:** one place that says what the lab *is right now*. Debriefs say what happened and what was learned; this says what exists. Update it in the same sitting as any change, and add a line to the change log at the bottom.

**Rules for this file:** never put passwords, setup keys, API tokens or recovery codes here. Record *where* a secret is kept, not the secret. Anything marked `(fill in)` or `(confirm)` is not verified yet.

Last verified: 2026-10-17 (partial — see markers)

---

## 1. Topology

```
Home router / ISP (inbound blocked)
        |
     vmbr0  (Proxmox WAN bridge)
        |
  [OPNsense VM]  --- WAN: vmbr0
        |
     vmbr1  (VLAN-aware trunk, no physical port)
        |-- VLAN 10  MGMT
        |-- VLAN 20  SERVERS      DC01, netbird-router
        |-- VLAN 30  WORKSTATIONS j2iq606, Ubuntu admin machine
        |-- VLAN 40  DMZ          (unused so far)

Remote access: devices --(outbound)--> NetBird cloud control plane
               <--direct/relay--> netbird-router (10.10.20.50) --> lab
```

Planned, not built (Part 9): `vmbr2` branch office with a second firewall; VLANs 60 VOICE, 70 GUEST, 80 IoT; VLAN 50 RED TEAM (Part 8).

## 2. Networks

| VLAN | Name | Subnet | Gateway (OPNsense) | DHCP | Notes |
|---|---|---|---|---|---|
| 10 | MGMT | 10.10.10.0/24 (confirm) | 10.10.10.1 (confirm) | (fill in) | |
| 20 | SERVERS | 10.10.20.0/24 | 10.10.20.1 | (fill in range) | Static: DC01 .10, netbird-router .50 |
| 30 | WORKSTATIONS | 10.10.30.0/24 (confirm) | 10.10.30.1 (confirm) | Kea (fill in range) | |
| 40 | DMZ | 10.10.40.0/24 (confirm) | 10.10.40.1 (confirm) | (fill in) | Reserved for reverse proxy / public-facing tests |

DNS: Unbound on OPNsense; AD DNS on DC01 (10.10.20.10) for `TermlessSolution.local`.

## 3. Hosts

| Host | Role | OS | VLAN / IP | Proxmox VM ID | vCPU / RAM / disk | Backup |
|---|---|---|---|---|---|---|
| Proxmox host | Hypervisor | Proxmox VE (fill in version) | (fill in) | n/a | (fill in) | n/a |
| OPNsense | Firewall / router / DHCP / DNS | OPNsense (fill in version) | GW on each VLAN; admin GUI 80→443 | (fill in) | (fill in) | Config export: (fill in date) |
| DC01 | AD DS + DNS, forest `TermlessSolution.local` | Windows Server 2025 | VLAN 20 / 10.10.20.10 static | (fill in) | (fill in) | None yet |
| j2iq606 | Domain-joined client (hostname desktop-j2iq606) | Windows | VLAN 30 / DHCP | (fill in) | (fill in) | None yet |
| netbird-router | NetBird routing peer | Ubuntu Server | VLAN 20 / 10.10.20.50 static (`ens18`, NetworkManager renderer) | (fill in) | (fill in) | None yet |
| Ubuntu admin machine | Opens OPNsense GUI, runs checks | Ubuntu | VLAN 30 | (fill in) | (fill in) | None yet |

## 4. Identity

- Forest/domain: `TermlessSolution.local`; DC01 is the only DC (no redundancy).
- OU structure, test users and groups: (fill in — export with `Get-ADOrganizationalUnit -Filter * | Select Name,DistinguishedName`).
- GPOs: Default Domain Policy (password/lockout), GPP drive mapping.
- Entra / M365: enterprise app for NetBird SSO requires user assignment (AADSTS50105 if not assigned). Tenant details: (fill in).

## 5. Firewall rules (summary)

| Interface | Rule | Status |
|---|---|---|
| SERVERS | Block TCP from SERVERS net to This Firewall, destination ports alias `OPNsense_GUI_Ports` (80, 443) | 🟡 Added 2026-10-13, **not yet effective** (scan still showed 80/443 open). Check rule order and Apply changes |
| All VLANs | Inter-VLAN pass/block rules | (fill in — export or screenshot the rule lists) |

Known exposure from scans (2026-10-13, from `netbird-router`): OPNsense GUI on 80/443 reachable from VLAN 20; DC01 exposes standard AD ports including WinRM 5985.

## 6. Remote access (NetBird)

- Control plane: hosted at app.netbird.io (self-hosting on Oracle free tier parked after capacity and region-quota blockers).
- Routing peer: `netbird-router`, Networks model, resource `10.10.0.0/16`.
- Groups: `homelab-routers`, `homelab-resources`, `remote-clients`. Policy: `remote-clients` → `homelab-resources`, currently **All protocols** (to be tightened).
- Enrolled: netbird-router, j2iq606, iPhone, offshore staff device (via SSO).
- Setup key: one was exposed in chat on 2026-10-10; **rotate/revoke it** (done? fill in).
- Secrets live in: (fill in — password manager name).

## 7. Open items

- [ ] Fix SERVERS block rule; re-scan (expect 53 open, 80/443 filtered)
- [ ] Tighten NetBird policy to specific ports/resources
- [ ] Revoke exposed setup key (confirm)
- [ ] Back up OPNsense config and test a restore (Part 9, scenario 6)
- [ ] Proxmox Backup Server and timed restores
- [ ] Wireshark exercise on j2iq606

## 8. Change log

| Date | Change | Why | Debrief |
|---|---|---|---|
| 2026-10-12 | NetBird routing peer working end-to-end | WireGuard blocked by ISP | `debrief-netbird-remote-access.md` |
| 2026-10-13 | Nmap scans of VLAN 20; SERVERS block rule for OPNsense GUI added | Reduce admin-plane exposure | pending |
| 2026-10-17 | This file created | No single current-state record existed | n/a |

## 9. Commands to fill in the blanks

On the Proxmox host:
```
qm list                        # VM IDs and names
qm config <vmid>               # CPU, RAM, disk, NIC bridge and VLAN tag
pveversion
```
In OPNsense: **System → Configuration → Backups → Download configuration** (store the file with your secrets, not in this repo). For rule lists: **Firewall → Rules → each interface**, screenshot with order visible.

On DC01 (PowerShell):
```
Get-ADOrganizationalUnit -Filter * | Select Name,DistinguishedName
Get-GPO -All | Select DisplayName,GpoStatus
Get-NetIPAddress -AddressFamily IPv4 | Select InterfaceAlias,IPAddress,PrefixOrigin
```
On Linux hosts: `ip -br a` and `sudo netplan get`.
