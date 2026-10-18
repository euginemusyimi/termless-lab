# Evidence Capture Checklist

A one-time catch-up of screenshots covering the lab from the start to now. After this, every change is documented the same day.

**Priority:** ⭐ = must have (carries a debrief or proves a claim). Unmarked = nice to have.

## How to capture

- **GUI screens → screenshot.** Windows: **Win+Shift+S**. Ubuntu: **Shift+PrtSc** (area) . Mac: **Cmd+Shift+4**. Crop to the relevant panel.
- **Terminal output → copy the text** instead of a screenshot (select, copy, paste into a text file). Text can be searched, redacted precisely and shown as a code block.
- **Name each file with its ID**, e.g. `B5b.png`, `E3.txt`. I rename them to the final convention (`docs/images/<topic>/<date>-<what>.png`).
- **Work one system at a time** (A, then B, and so on) so you stay logged in and in one window.
- **Redact:** public IP, setup keys, tokens, tenant/subscription IDs, email addresses, QR codes, WireGuard private keys. If unsure, send it unredacted to me here and I will flag it. Do not post it publicly first.

## A. Proxmox (web UI)

| ID | Capture | Used in |
|---|---|---|
| A1 ⭐ | Node → **System → Network**: vmbr0 and vmbr1, "VLAN aware" visible | As-built, VLAN debrief |
| A2 ⭐ | Left tree showing all VMs with names and IDs | As-built |
| A3a–d ⭐ | Each VM's **Hardware** tab, NIC row with the VLAN tag visible (OPNsense, DC01, j2iq606, netbird-router) | VLAN tag debrief |
| A4 | Node **Summary** (version, CPU, RAM) | As-built |

## B. OPNsense (web UI)

| ID | Capture | Used in |
|---|---|---|
| B1 ⭐ | **Interfaces → Assignments** | As-built |
| B2 ⭐ | **Interfaces → Other Types → VLAN** list | As-built |
| B3 | **Services → Kea DHCP → Leases** (and the subnet list) | dnsmasq / Kea debrief |
| B4 | **Services → Unbound DNS → General** | As-built |
| B5a–d ⭐ | **Firewall → Rules** for MGMT, SERVERS, WORKSTATIONS, DMZ, rule order and descriptions visible | Segmentation debrief |
| B6 ⭐ | **Firewall → Aliases**, `OPNsense_GUI_Ports` open with 80 and 443 | Segmentation debrief |
| B7 | **Firewall → NAT → Outbound** | As-built |
| B8 | **System → Firmware → Status** (version) | As-built |
| B9 | **VPN → WireGuard** instance/peers page (hide private keys) | WireGuard debrief |
| B10 ⭐ | **Firewall → Log Files → Live View** showing a block line (after the rule fix) | Segmentation debrief |
| B11 | **Lobby → Dashboard** | Home page / as-built |

## C. DC01 (Active Directory)

| ID | Capture | Used in |
|---|---|---|
| C1 | **Server Manager** dashboard with AD DS and DNS roles | As-built |
| C2 ⭐ | **Active Directory Users and Computers**, OU tree expanded | As-built |
| C3 ⭐ | **Group Policy Management**, forest tree with GPOs | As-built |
| C4 | Default Domain Policy → password and lockout settings | As-built |
| C5 | The drive-mapping preference item | As-built |
| C6 | **DNS Manager**, forward zone `TermlessSolution.local` | As-built |
| C7 ⭐ | Text: `Get-NetIPAddress -AddressFamily IPv4 \| Select InterfaceAlias,IPAddress,PrefixOrigin` | VLAN tag debrief (static vs DHCP) |
| C8 | Text: `Get-ADDomain \| Select DNSRoot,DomainMode,PDCEmulator` | As-built |

## D. Client `j2iq606`

| ID | Capture | Used in |
|---|---|---|
| D1 ⭐ | Text: `systeminfo \| findstr /i "domain host"` | As-built |
| D2 ⭐ | Text: `gpresult /r` (applied GPOs) | As-built |
| D3 | File Explorer showing the mapped drive | As-built |
| D4 | NetBird client status window or `netbird status -d` text | NetBird debrief |

## E. `netbird-router` (Ubuntu terminal, all as text)

| ID | Capture | Used in |
|---|---|---|
| E1 ⭐ | `ip -br a` | netplan debrief, as-built |
| E2 ⭐ | `sudo netplan get` | netplan debrief |
| E3 ⭐ | `sudo netbird status -d` | NetBird debrief |
| E4 | `cat /proc/sys/net/ipv4/ip_forward` | NetBird debrief |
| E5 | `sudo nmap -sn 10.10.20.0/24` and the DC01 scan | Segmentation debrief |

The 13 Oct Nmap output is already with me, so E5 only needs re-running after the block rule works.

## F. NetBird dashboard and Entra

| ID | Capture | Used in |
|---|---|---|
| F1 ⭐ | **Peers** page (names, status, groups) | NetBird debrief |
| F2 ⭐ | **Networks → homelab**: resource and routing peer | NetBird debrief |
| F3 ⭐ | **Access Control → Policies**: the `remote-clients` policy | NetBird debrief |
| F4 | **Groups** | NetBird debrief |
| F5 | **Settings → Setup Keys** list (names and status only, never key values) | NetBird debrief |
| F6 | Entra → **Enterprise applications → NetBird → Users and groups** (assignment) | NetBird debrief |
| F7 | The AADSTS50105 error page, if you still have it | NetBird debrief |

## G. Phone

| ID | Capture | Used in |
|---|---|---|
| G1 ⭐ | NetBird app connected with Wi-Fi off (cellular only) | NetBird debrief |
| G2 | The phone reaching a lab resource | NetBird debrief |

## H. Evidence you may already have

Dig these out of your earlier chats, camera roll or downloads. They are the most valuable because they show failures that are now fixed:

- Oracle "Out of capacity" and region-quota errors
- The failing gateway ping and "Destination Host Unreachable"
- NetBird status showing Management disconnected
- The canyouseeme.org result from the WireGuard test
- The Proxmox or OPNsense screen from the kernel-panic reinstall, or the NO-CARRIER message

Send these as they are. If an old failure can't be recovered, **recreate it on purpose** in a throwaway VM (for example set a wrong VLAN tag and ping the gateway). That doubles as the variation test in the debrief.

## Status

| Batch | Sent | Placed in repo |
|---|---|---|
| A Proxmox | ☐ | ☐ |
| B OPNsense | ☐ | ☐ |
| C DC01 | ☐ | ☐ |
| D j2iq606 | ☐ | ☐ |
| E netbird-router | ☐ | ☐ |
| F NetBird / Entra | ☐ | ☐ |
| G Phone | ☐ | ☐ |
| H Old failures | ☐ | ☐ |
