# Lab Task Debrief

## 0. Task identifier

**Date:** (MSP Homelab build — Phase 3, WireGuard VPN attempt)
**Task/lab name:** WireGuard remote access setup — inbound connectivity blocked upstream of home network
**Related project:** MSP Homelab (Part 1) — Phase 3 VPN
**Time spent:** Multi-session (config build, client setup across desktop + iOS, dual-stack IPv4/IPv6 investigation, port-forwarding, ISP-blocking isolation)

---

## 1. What was I trying to do?

Stand up a working WireGuard VPN on OPNsense so a remote client (outside the home network, on cellular data) could tunnel in and reach internal VLANs (SERVERS/AD, etc.) — demonstrating the "VPNs" required skill from the target job postings.

---

## 2. What did I actually do?

```
# OPNsense side
- Installed os-wireguard plugin
- Created Local instance (wg0): tunnel address 10.10.50.0/24, DNS 10.10.20.10
- Created Peer entry, generated keypair
- Enabled WireGuard service, assigned + enabled the wg0 interface
- Firewall rules: WAN (UDP/51820 initially "any/any" by mistake, later scoped
  via alias), WireGuard interface (pass any)

# Client side
- Desktop (Windows): built wg0.conf manually, imported into WireGuard app
- iOS: separately generated/imported config, resulted in an IPv6 endpoint
  mismatch vs. the desktop's IPv4 config — surfaced the dual-stack issue

# Diagnostic chain (this is the real content of the task)
Get-NetTCPConnection / WireGuard "Transfer" stats  → 0 B received, sent only
  → ruled out: OPNsense firewall rule (widened to any/any, still failed)
  → ruled out: TP-Link port forward (confirmed correct via screenshot: enabled,
    UDP, correct internal IP 192.168.0.37, correct port both sides)
  → ruled out: CGNAT (TP-Link's own WAN IP matched external curl -4 ifconfig.me
    exactly — genuinely public, uniquely-owned IPv4)
  → ruled out: wrong port targeted by ISP filtering specifically (changed
    listen port to an alternate, re-tested via canyouseeme.org — still failed)
  → confirmed: ISP is blocking unsolicited inbound traffic generally,
    upstream of all owned equipment
```

---

## 3. What broke — on purpose or not?

**Broke on purpose? (Y/N):** N — the VPN never worked in the first place; this was pure real-world troubleshooting, not injected fault.

**What was the failure signature?** WireGuard client showed `Data sent` increasing steadily (handshake retries) but `Data received: 0 B`, indefinitely. No error message anywhere — UDP is connectionless, so a silently dropped/blocked packet produces no feedback at all, just an endlessly retrying client.

**How did I diagnose it?** Worked outward from the client, methodically eliminating each layer:
1. Verified config correctness (endpoint, keys) on both desktop and iOS clients — caught a real dual-stack (IPv4 vs IPv6) mismatch between the two clients along the way
2. Verified OPNsense's own WAN IP was a private DHCP-assigned address (`192.168.0.37`), confirming a double-NAT topology (home router → OPNsense)
3. Verified/corrected the OPNsense WAN firewall rule (caught two real mistakes here: initially "any/any" instead of scoped, and later a wrong `Source: WAN address` value that would never match real client traffic)
4. Verified the home router's port-forward rule via direct screenshot evidence
5. Used `canyouseeme.org` as an external ground-truth test, independent of any of my own equipment
6. Ruled out CGNAT by comparing the router's self-reported WAN IP against an external site's view of it
7. Ruled out port-specific ISP filtering by testing a second, less-common port

**Root cause:** ISP-level inbound traffic blocking, upstream of the home router entirely — not fixable via any local configuration change.

**Fix:** No local fix exists for the original goal (direct home-to-client WireGuard). Documented as a hard external constraint. Real fixes going forward: (a) a cloud VPS acting as a relay, since outbound connections are never blocked, or (b) rely on Tailscale, which already works because it uses NAT-traversal/relay techniques designed for exactly this scenario.

---

## 4. What would I do differently next time?

- **Run the `canyouseeme.org` (or equivalent external reachability) test first**, before spending time on client-side config correctness. It's the single test that would have told me "this is an external/ISP problem" in five minutes, rather than discovering it only after ruling out every local possibility one at a time.
- When two clients (desktop vs. iOS) show different endpoint types (IPv4 vs IPv6) for what should be the same config, treat that as an immediate red flag to investigate the network's dual-stack behavior early, rather than a side detail to note and move past.
- Fix obviously-wrong firewall rule fields (like `Source: WAN address` on an inbound WAN rule) the moment they're spotted, rather than leaving them for later — they add noise to the diagnostic process even when they aren't the actual root cause.

---

## 5. Real-world context

**Link:** Search "ISP blocking inbound VPN ports residential" / "CGNAT vs ISP port blocking WireGuard" — this is a widely documented, common problem for anyone self-hosting a VPN, mail server, or any inbound service on residential internet. Many home-lab and self-hosting communities (r/selfhosted, r/homelab) have extensive threads specifically about this exact symptom (0 bytes received, config verified correct, still failing) with ISP-level blocking as the resolution in a large fraction of cases.

**Connection:** This mirrors a real architectural decision MSPs and remote-work-enabling IT teams face constantly: not every client site has straightforward, unrestricted inbound internet access (ISP policy, business-tier vs residential-tier plans, carrier-grade NAT on cellular/some fiber providers). The standard real-world answer is exactly what's proposed here — a cloud relay/bastion host that clients dial into, since outbound-initiated connections almost always work even when inbound is blocked. This is the same underlying pattern as SD-WAN's centralized-controller model (Module 4b) and cloud VPN gateways (Module 1a) — a third party mediates connectivity rather than relying on direct site-to-site inbound reachability.

---

## 6. Explain it to a junior colleague

When you set up a VPN server at home, your device needs to accept inbound connections from the internet. But your home internet almost always sits behind at least one layer of address translation (your router), and sometimes your ISP does extra filtering on top of that — blocking connections that come in unsolicited, even if you've correctly configured everything on your own equipment. The tricky part is that this kind of block is usually silent: there's no error message, the connection just times out forever, which looks identical to a misconfiguration. The way to tell the difference is to test from a completely independent, external vantage point — a website or tool that isn't part of your own network at all — to see whether the packet ever arrives at your router's front door in the first place. If it doesn't, no amount of fixing your own router or firewall will help, because the problem is happening further upstream, at your internet provider, before your traffic ever reaches your house. The practical fix in that situation is to flip the direction: instead of waiting for the outside world to connect in, you have something inside your network reach *out* to a server that does have a public, unrestricted address — outbound connections almost always work even when inbound ones are blocked.

---

## 7. Variation test

**Variable changed:** Attempted an alternate listen port (`51820` → alternate port) to distinguish "specific port blocked" from "all inbound blocked."
**Result:** Both ports failed identically via `canyouseeme.org` — confirms general inbound blocking rather than port-specific filtering. This *is* the variation test for this task; a further one worth running later: test with the VPS-relay approach (Option 1) to confirm outbound-initiated tunnels succeed where inbound-initiated ones failed.

---

## 8. Tags

`#VPN` `#WireGuard` `#OPNsense` `#NAT` `#double-NAT` `#port-forwarding` `#CGNAT` `#ISP-blocking` `#IPv4-IPv6-dual-stack` `#firewall-rule-misconfiguration` `#external-reachability-testing` `#canyouseeme` `#diagnostic-methodology` `#cloud-relay-pattern`
