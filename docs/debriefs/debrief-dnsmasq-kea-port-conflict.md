# Lab Task Debrief

## 0. Task identifier

**Date:** (retroactive — pulled from earlier session of MSP Homelab build)
**Task/lab name:** OPNsense Kea DHCP not listening on port 67 — dnsmasq service conflict
**Related project:** MSP Homelab (Part 1) — OPNsense VLAN/DHCP rebuild post-reinstall
**Time spent:** ~1 session (diagnosis + fix + retest across 4 VLANs)

---

## 1. What was I trying to do?

Get Kea DHCPv4 serving leases correctly across all four VLANs (MGMT/SERVERS/WORKSTATIONS/DMZ) after a full OPNsense reinstall following a kernel-panic corruption. `configctl kea status` reported the service as running, but test clients weren't getting DHCP leases.

---

## 2. What did I actually do?

```
# Confirmed Kea "looked" healthy
configctl kea status
# → showed running, PID present

# Actual ground-truth check — what's really bound to port 67
sockstat -4 -p 67
# → revealed dnsmasq bound to *:67, not Kea

# Fix: disable dnsmasq's Enable checkbox (unified DNS+DHCP toggle in this
# OPNsense version — no separate DHCP-only disable)
# Services → Dnsmasq DNS & DHCP → General → uncheck Enable → Apply

# Restart Kea and re-verify port ownership
configctl dnsmasq restart
configctl kea restart
sockstat -4 -p 67
# → confirmed Kea now owns *:67

# Watched a live lease negotiation to confirm actual service, not just port ownership
tail -f /var/log/kea/latest.log

# Retested from Ubuntu client VM
sudo nmcli device connect ens18
ping 10.10.10.1
ping 8.8.8.8
```

---

## 3. What broke — on purpose or not?

**Broke on purpose? (Y/N):** N — this was a real post-reinstall misconfiguration, discovered while trying to get DHCP working, not deliberately injected.

**What was the failure signature?** No obvious error anywhere. `configctl kea status` reported "running" with a valid PID. No crash, no failure message. The only symptom was client VMs failing to get a DHCP lease — a silent failure with a healthy-looking service status.

**How did I diagnose it?** Started from the symptom (no lease) rather than trusting the service status. Went straight to socket-level ground truth with `sockstat -4 -p 67` instead of continuing to trust `configctl kea status`. That immediately showed dnsmasq, not Kea, actually bound to the port — a completely different signal than "is the process alive."

**Root cause:** Two DHCP-capable services (Kea and legacy dnsmasq) were both enabled on the box after the reinstall. Whichever grabbed port 67 first won the socket; the other kept running as a process but was functionally inert for DHCP purposes. Kea's own "Firewall rules" auto-checkbox only permits the DHCP protocol itself through the firewall — it does nothing about a port-binding conflict with another service.

**Fix:** Disabled dnsmasq entirely (this OPNsense version has no split DNS/DHCP toggle — one Enable checkbox governs both), restarted Kea, confirmed via `sockstat` that Kea now owned the port, then confirmed via live log tail that it was actually negotiating leases (DISCOVER/OFFER/REQUEST/ACK), not just holding the socket.

---

## 4. What would I do differently next time?

- Check `sockstat -4 -p 67` (or the Linux/other-OS equivalent, e.g. `ss -ulnp` / `netstat`) as a **first-line diagnostic**, not a last resort, whenever a network service "looks running" but isn't behaving as expected. "Service is running" and "service owns the resource it needs" are two different questions, and the second one is usually the real one.
- After any fresh OS/appliance reinstall, explicitly check for other services on the box that could compete for the same ports (DNS/DHCP tools especially tend to ship enabled-by-default and silently coexist) before assuming a single-service config problem.
- Don't stop at "port ownership fixed" — confirm with a live log tail that the service is actually completing full transactions, not just holding the socket. A bound port doesn't guarantee correct behavior.

---

## 5. Real-world context

**Link:** ISC Kea DHCP documentation explicitly warns about port-binding conflicts with other DHCP daemons (ISC DHCP, dnsmasq) running on the same host — this is a documented, common gotcha in real Kea deployments, not a one-off homelab quirk. (Search "Kea DHCP port 67 already in use dnsmasq" for current community/vendor threads — this exact conflict is reported repeatedly across OPNsense/pfSense forums post-migration from ISC DHCP to Kea.)

**Connection:** This is the same class of failure as a "silent service degradation" incident — a monitoring system checking only "is the process running" (like an early version of `configctl kea status`) would report green/healthy while DHCP was completely non-functional for real clients. This is exactly why production monitoring for critical services should check functional behavior (does it actually issue a lease / answer a query) rather than just process liveness — a lesson that generalizes well beyond DHCP to any service health check.

---

## 6. Explain it to a junior colleague

When two services both try to listen on the same network port, only one of them actually gets it — whichever grabbed it first. The other one can still be "running" as a process, show up in your service manager, even have a valid process ID, but it's functionally dead for that job because the operating system won't let two things listen on the same port at once. So "the service says it's running" and "the service is actually doing its job" are different questions, and you have to check both. The tool to check "who actually owns this port" is a socket-status command like `sockstat` (BSD/OPNsense) or `ss`/`netstat` (Linux) — it shows you the real, ground-truth picture of what's bound to what, cutting through whatever the service manager is reporting.

---

## 7. Variation test

**Variable changed:** *(not yet run — candidate for next pass: deliberately re-enable dnsmasq's DHCP alongside Kea on a test VLAN, confirm the same conflict reproduces, then try the reverse — starting dnsmasq second, after Kea already owns the port — to see how the failure signature changes when the ordering flips.)*
**Result:** *(pending)*

---

## 8. Tags

`#DHCP` `#Kea` `#dnsmasq` `#OPNsense` `#port-conflict` `#service-health-vs-liveness` `#socket-diagnostics` `#sockstat` `#silent-failure` `#post-reinstall-config` `#VLAN-DHCP`
