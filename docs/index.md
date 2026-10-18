# Termless Lab

**Field notes from a Proxmox + OPNsense lab for MSP and systems administration work.**

A hands-on lab built to practise and demonstrate systems and network administration: segmented VLANs, Active Directory, remote access without inbound ports, and the troubleshooting that went with each.

## Start here

- **[As-built record](as-built.md)**: what the lab is right now
- **[Debriefs](debriefs/index.md)**: real incidents, root causes and lessons
- **[Concept map](concept-map.md)**: the ideas behind them, indexed by concept
- **[Roadmap](roadmap.md)**: what is done, in progress and planned
- **[Screenshot guide](reference/screenshots.md)**: how evidence is captured and redacted

## Highlights

- Segmented network (MGMT / SERVERS / WORKSTATIONS / DMZ) on OPNsense with Kea DHCP
- Windows Server AD DC, domain-joined clients, GPOs
- NetBird mesh VPN with a routing peer, working after the ISP blocked WireGuard
- Port scans (Nmap) used to check what the firewall rules actually expose

*Lab only. It holds no client data or production systems. No credentials are published here.*
