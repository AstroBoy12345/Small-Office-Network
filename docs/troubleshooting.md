# Troubleshooting Log

Real problems found while building this lab, how they were diagnosed and fixed.

---

## 1. Native VLAN mismatch while configuring trunks

**Symptom**
```
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on GigabitEthernet0/2 (999), with SW2 GigabitEthernet0/1 (1).
```

**Cause** – SW1's trunk was already set to native VLAN 999 while SW2 was still using VLAN 1.
Spanning Tree put the port into an inconsistent (blocked) state to stop traffic leaking between VLANs.

**Fix** – configured `switchport trunk native vlan 999` on SW2. The log confirmed recovery:
```
%SPANTREE-2-UNBLOCK_CONSIST_PORT: Unblocking GigabitEthernet0/1 on VLAN0999. Port consistency restored.
```

**Lesson** – both ends of a trunk must agree on the native VLAN.

---

## 2. A port stayed in VLAN 1 after an `interface range` command

**Symptom** – `show vlan brief` on SW2 still listed `Gig0/2` under VLAN 1, although it should be parked in VLAN 666.

**Cause** – the combined range `interface range FastEthernet0/21 - 24 , GigabitEthernet0/2` was only partly applied: Fa0/21–24 moved to VLAN 666, Gi0/2 did not.

**Fix** – configured Gi0/2 on its own (`switchport mode access`, `switchport access vlan 666`, `shutdown`).

**Lesson** – when a command fails partway, IOS applies what it understood. Always verify with a `show` command.

---

## 3. No internet: WAN and NAT commands entered on the wrong router

**Symptom** – `ping 8.8.8.8` from R1-EDGE failed. `show ip route` showed `Gateway of last resort is not set` and no 203.0.113.0/30 network.

**Diagnosis**
- `show ip interface brief` on the ISP showed **203.0.113.2** on G0/0 (R1's address) and subinterfaces G0/1.10, .20, .30.
- The WAN, default route and NAT commands had been pasted into the **ISP** instead of R1-EDGE.
- Because R1's G0/0 was never configured, the next hop 203.0.113.1 was unreachable. A static route is only installed when its next hop is reachable (recursive lookup), so the default route was missing too.

**Fix** – erased and reloaded the ISP, reapplied its correct config, then configured WAN, default route and PAT on R1-EDGE.

**Lesson** – check the prompt before pasting configuration.

---

## 4. DNS and web page failing: IP address typo on the server

**Symptom** – `nslookup www.example.com` timed out and the web page did not load.

**Diagnosis (hop by hop)**
1. ISP → `ping 198.51.100.10` **failed**, so the problem was on the last link.
2. ISP `show ip interface brief`: G0/1 up/up with the correct IP, so the ISP side was fine.
3. Server `ipconfig`: IPv4 address **192**.51.100.10 instead of **198**.51.100.10.
   The ISP's ARP requests for 198.51.100.10 were never answered.

**Second issue** – after correcting it, `ipconfig` still showed the old address. In Packet Tracer the change is only applied when focus leaves the field (Enter/Tab).

**Fix** – set the address in *Config → FastEthernet0* and pressed Enter. All tests then passed.

**Lesson** – test one hop at a time from the closest device outwards, and verify that a change actually took effect.

---

## Notes

- `show vlan` on a router shows the local VLAN database. The command for router-on-a-stick is `show vlans`.
- `sl_def_acl` in `show access-lists` on the 2911 is a built-in default ACL. It is not applied to any interface.
- Refused SSH attempts from non-admin subnets do not count as login failures for `login block-for`, because the VTY `access-class` rejects them before the login prompt.
