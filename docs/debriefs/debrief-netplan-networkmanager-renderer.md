# Lab Task Debrief — Static IP ignored on Ubuntu (netplan wrong interface name + NetworkManager renderer)

## 0. Task identifier

**Date:** 2026-09-27
**Task/lab name:** netbird-router static IP not applying (netplan `eth0` vs `ens18`, NetworkManager renderer)
**Related project:** NetBird routing peer VM (Part 1 VPN row / Part 6 Phase 8)
**Time spent:** (fill in — roughly 40 minutes of back-and-forth)

---

## 1. What was I trying to do?

Give the new Ubuntu VM a static address (`10.10.20.50/24`, gateway `10.10.20.1`, DNS `10.10.20.10`) so it doesn't wander like DC01 did.

---

## 2. What did I actually do?

```
sudo nano /etc/netplan/00-installer-config.yaml     # wrote static config
sudo netplan apply
ip a                       # still showed 10.10.20.102 ... dynamic  (DHCP, not static)
cat /etc/netplan/00-installer-config.yaml   # config named interface eth0
ip a                       # actual interface name: ens18
# fixed eth0 -> ens18, re-applied: still DHCP
sudo netplan get           # config registered correctly, but "renderer: NetworkManager"
ls -la /etc/netplan/       # 00-installer-config.yaml AND 01-network-manager-all.yaml
nmcli connection show      # netplan-ens18 (active) and "Wired connection 1" (inactive)
nmcli device status        # ens18 connected, profile netplan-ens18
sudo nmcli connection up netplan-ens18     # static address finally applied
ip a                       # 10.10.20.50/24, no "dynamic" flag
```

---

## 3. What broke — on purpose or not?

**Broke on purpose? (Y/N):** N

**What was the failure signature?** `netplan apply` appeared to succeed (with a warning: `systemd-networkd is not running ... Unit dbus-org.freedesktop.network1.service not found`), but `ip a` still showed the DHCP lease marked `dynamic`.

**How did I diagnose it?** Compared what netplan *thought* it applied (`netplan get`) with what the interface actually had (`ip a`). Dead end 1: the YAML named `eth0`, which doesn't exist on this VM, and netplan silently ignores config for a nonexistent interface. Dead end 2: after fixing the name, the config was registered but still not live. `netplan get` revealed the renderer was **NetworkManager** (set by `01-network-manager-all.yaml`), not systemd-networkd, so the networkd warning was a red herring.

**Root cause:** Two stacked problems. (1) Wrong interface name in the YAML. (2) With NetworkManager as the renderer, the regenerated `netplan-ens18` profile was not re-activated on the already-connected interface, so the old DHCP state stayed live until the profile was explicitly brought up. The exact reason NetworkManager didn't re-apply it on its own is **not fully established**.

**Fix:** Correct the interface name, then `sudo nmcli connection up netplan-ens18`.

---

## 4. What would I do differently next time?

- Run `ip -br a` **before** writing any network config, and copy the interface name from it.
- After `netplan apply`, always compare `ip a` against intent; "no error" isn't proof of success.
- Run `sudo netplan get` early to see which renderer is in control, and read warnings about a service (networkd) in light of that.
- Use `sudo netplan try` for remote sessions so a bad config rolls back.

---

## 5. Real-world context

Named, public incident for this exact pattern: **not found yet.** Generic pattern: config that is accepted without error but never takes effect, discovered only by checking actual state.

**Link:** (to find)
**Connection:** Same family as the earlier "service shows running but isn't doing the job" lesson (Kea/dnsmasq) — verify real state, not the tool's say-so.

---

## 6. Explain it to a junior colleague

On Ubuntu, netplan is just a translator: you describe the network in YAML and it hands the settings to a renderer, either systemd-networkd or NetworkManager. Two things can make your static IP silently not apply: naming an interface that doesn't exist (netplan ignores it without complaint), or the renderer not re-applying to an already-active connection. Always check the real interface name with `ip a`, use `netplan get` to see the renderer, and confirm with `ip a` that the address no longer says `dynamic`.

---

## 7. Variation test (do this on a later pass, not required same-day)

**Variable changed:** (proposed, not yet done) Set `renderer: networkd` explicitly on a fresh Ubuntu server VM and repeat the static-IP change; compare what `netplan apply` does vs the NetworkManager case.
**Result:** (pending)

---

## 8. Tags

`#netplan` `#NetworkManager` `#systemd-networkd` `#static-vs-DHCP-addressing` `#silent-failure` `#verify-actual-state` `#interface-naming`
