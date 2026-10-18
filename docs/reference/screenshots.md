# Screenshot Guide

Screenshots turn a claim into evidence. Add them next to the text that makes the claim, not in a separate gallery.

## Where files go

```
docs/images/<topic>/<yyyy-mm-dd>-<what-it-shows>.png
e.g. docs/images/netbird/2026-10-12-peers-connected.png
```

Topics: `proxmox`, `opnsense`, `active-directory`, `netbird`, `nmap`, `wireshark`. Use PNG, keep width under about 1600 px, and one idea per image.

## How to embed

```markdown
![NetBird dashboard showing the routing peer and phone connected](../images/netbird/2026-10-12-peers-connected.png){ width="700" }
*Figure: both peers show Connected after the Networks policy was added.*
```

Always write alt text that says what the image proves. From a debrief the path starts with `../../images/`; from the top-level pages, `images/`.

## Redact before saving

Public IP address, setup keys and tokens, tenant and subscription IDs, email addresses, QR codes, session cookies, and anything with client or employer data. Black boxes beat blur. Re-open the saved file to check. If something sensitive was ever visible, treat it as exposed.

## Shot list

| Topic | Capture | Goes in |
|---|---|---|
| proxmox | Datacenter → host → Network (vmbr0, vmbr1, VLAN aware ticked) | As-built |
| proxmox | Each VM's Hardware tab with the NIC VLAN tag visible | VLAN tag debrief |
| opnsense | Interfaces → VLANs list | As-built |
| opnsense | Firewall → Rules for SERVERS, order visible, before and after the fix | Segmentation debrief |
| opnsense | Firewall → Log Files → Live View showing the block line | Segmentation debrief |
| opnsense | Kea DHCP leases / service status | dnsmasq / Kea debrief |
| active-directory | ADUC OU tree; `gpresult` output; domain join confirmation | As-built |
| netbird | Peers page, Networks resource, access policy | NetBird debrief |
| netbird | `netbird status -d` output, phone connected on cellular | NetBird debrief |
| netbird | AADSTS50105 error and the fix (enterprise app assignment) | NetBird debrief |
| nmap | Terminal output before and after the block rule | Segmentation debrief |
| wireshark | DNS, Kerberos and LDAP captures on j2iq606 with filters visible | Concept map / new debrief |

Prefer a screenshot of the *failure* and the *fix* together over a screenshot of a finished screen.
