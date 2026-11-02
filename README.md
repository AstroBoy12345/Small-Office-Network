# Small Office / Branch Network – Cisco Packet Tracer

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?logo=cisco&logoColor=white)
![Level](https://img.shields.io/badge/Level-CCNA-blue)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

A secure two-floor small-office network, designed, configured and tested in Cisco Packet Tracer.
It covers the core CCNA skills: **VLAN segmentation, trunking, inter-VLAN routing, DHCP, PAT,
ACL-based guest isolation, Layer 2 security and SSH-only management.**

![Topology](NetworkImages/topology.png)
<sub>Click the diagram to view it full size.</sub>

## Scenario

A small company with two floors needs a network where:

- **Staff** have internal access, IP phones and internet
- **Guests** get internet only, with no access to anything internal
- **IT** can manage every network device securely and remotely
- **Network devices** cannot reach the internet themselves

## Features

| Area | What is implemented |
|---|---|
| **Segmentation** | VLANs 10 STAFF, 20 VOICE, 30 GUEST, 99 MGMT, 666 BLACKHOLE, 999 NATIVE · VTP transparent |
| **Trunking** | 802.1Q, native VLAN 999, allowed-VLAN list, DTP disabled (`nonegotiate`) |
| **Routing** | Router-on-a-stick (subinterfaces) · static default route to the ISP |
| **Services** | DHCP server on the router (per-VLAN pools, exclusions, option 150 for phones) · DNS/HTTP server on the "internet" |
| **Internet access** | PAT (NAT overload) – all users share one public IP |
| **Guest isolation** | Extended ACL – guests reach the internet only, all private (RFC 1918) ranges blocked |
| **Layer 2 security** | Port security (sticky MAC), BPDU Guard, PortFast, unused ports shut down in a blackhole VLAN |
| **Spanning Tree** | Rapid PVST+ with a planned root bridge (SW1 primary, SW2 backup) |
| **Management** | Dedicated MGMT VLAN · SSH v2 only · VTY access limited by ACL · login brute-force protection · password policy |

## Devices

| Device | Model | Role |
|---|---|---|
| R1-EDGE | Cisco 2911 | Office gateway – inter-VLAN routing, DHCP, NAT, ACLs |
| SW1 | Cisco 2960-24TT | Floor 1 switch, STP root bridge |
| SW2 | Cisco 2960-24TT | Floor 2 switch (staff + guest area) |
| ISP | Cisco 2911 | Simulated internet provider |
| SRV-INTERNET | Server-PT | Public DNS + web server (`www.example.com`) |
| PC-STAFF1/2, PRINTER, PC-ADMIN, LAPTOP-GUEST | End devices | Test clients |

## Addressing summary

| VLAN | Name | Subnet | Gateway | Addressing |
|---|---|---|---|---|
| 10 | STAFF | 192.168.10.0/24 | 192.168.10.1 | DHCP (.21 and up) |
| 20 | VOICE | 192.168.20.0/24 | 192.168.20.1 | DHCP (.21 and up) |
| 30 | GUEST | 192.168.30.0/24 | 192.168.30.1 | DHCP (.21 and up) |
| 99 | MGMT | 192.168.99.0/24 | 192.168.99.1 | Static |
| – | WAN | 203.0.113.0/30 | 203.0.113.1 (ISP) | Static |

➡️ Full IP plan, switch port map and cabling: [docs/ip-addressing-plan.md](docs/ip-addressing-plan.md)
