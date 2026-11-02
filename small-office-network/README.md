# Small Office / Branch Network – Cisco Packet Tracer

A secure two-floor small-office network built and tested in Cisco Packet Tracer.
It covers the core CCNA skills: VLAN segmentation, trunking, inter-VLAN routing (router-on-a-stick),
DHCP, PAT to an ISP, guest isolation with ACLs, Layer 2 security and SSH-only management.

![Topology](images/topology.png)

## Scenario

A small company has two floors. The network must give:

- **Staff** – internal access, IP phones and internet
- **Guests** – internet only, no access to anything internal
- **IT** – secure remote management of every network device
- **Management devices** – no internet access at all

## Features

| Area | What is implemented |
|---|---|
| Segmentation | VLANs 10 STAFF, 20 VOICE, 30 GUEST, 99 MGMT, 666 BLACKHOLE, 999 NATIVE · VTP transparent |
| Trunking | 802.1Q, native VLAN 999, allowed-VLAN list, DTP disabled (`nonegotiate`) |
| Routing | Router-on-a-stick (subinterfaces), static default route to ISP |
| Services | DHCP server on the router (per-VLAN pools, exclusions, option 150 for phones) · DNS/HTTP server on the "internet" |
| Internet access | PAT (NAT overload) – all users share one public IP |
| Guest isolation | Extended ACL: guests reach the internet only, all RFC 1918 ranges blocked |
| Layer 2 security | Port security (sticky MAC, restrict/shutdown), BPDU Guard, PortFast, unused ports shut down in a blackhole VLAN |
| Spanning Tree | Rapid PVST+, root bridge chosen on purpose (SW1 primary, SW2 backup) |
| Management | Dedicated MGMT VLAN + SVIs, SSH v2 only, VTY access restricted by ACL, login brute-force protection, password policy |

## Devices

| Device | Model | Role |
|---|---|---|
| R1-EDGE | Cisco 2911 | Office gateway: inter-VLAN routing, DHCP, NAT, ACLs |
| SW1 | Cisco 2960-24TT | Floor 1 switch, STP root bridge |
| SW2 | Cisco 2960-24TT | Floor 2 switch (staff + guest area) |
| ISP | Cisco 2911 | Simulated internet provider (Loopback 8.8.8.8) |
| SRV-INTERNET | Server-PT | Public DNS + web server (`www.example.com`) |
| PC-STAFF1/2, PRINTER, PC-ADMIN, LAPTOP-GUEST | End devices | Test clients |

## Addressing summary

| VLAN | Name | Subnet | Gateway | Assignment |
|---|---|---|---|---|
| 10 | STAFF | 192.168.10.0/24 | 192.168.10.1 | DHCP (.21+) |
| 20 | VOICE | 192.168.20.0/24 | 192.168.20.1 | DHCP (.21+) |
| 30 | GUEST | 192.168.30.0/24 | 192.168.30.1 | DHCP (.21+) |
| 99 | MGMT | 192.168.99.0/24 | 192.168.99.1 | Static |
| – | WAN | 203.0.113.0/30 | 203.0.113.1 (ISP) | Static |
| – | Internet LAN | 198.51.100.0/24 | 198.51.100.1 | Static |

Full plan, port map and cabling: [docs/ip-addressing-plan.md](docs/ip-addressing-plan.md)

## Design decisions

- **Router-on-a-stick** – one trunk link handles routing for all VLANs; right-sized for a small office.
- **VTP transparent** – avoids the risk of a switch with a higher revision number wiping the VLAN database.
- **Native VLAN 999 (no users)** – removes VLAN 1 from trunks and prevents VLAN hopping via double tagging.
- **Blackhole VLAN 666** – unused ports are shut down *and* placed in a VLAN that is not allowed on any trunk.
- **MGMT VLAN excluded from NAT** – network devices have no reason to reach the internet.
- **Guest ACL applied inbound on the guest subinterface** – extended ACLs go as close to the source as possible.
- **Different port-security modes** – `restrict` on staff ports (log, keep the port up), `shutdown` on guest and admin ports (strict).

## Verification

Every feature was tested. Results are documented in [docs/verification.md](docs/verification.md).

| Test | Result |
|---|---|
| Staff PC gets DHCP address, reaches gateway, other floor, printer | ✅ |
| Staff and guest browse `www.example.com` (DNS + HTTP through PAT) | ✅ |
| Guest pings staff PC, switch or staff gateway | ❌ blocked (16 ACL deny matches) |
| PC-ADMIN connects via SSH to R1, SW1, SW2 | ✅ |
| Staff PC tries SSH to network devices | ❌ refused |
| PC-ADMIN reaches the internet | ❌ by design (MGMT not translated) |

![NAT translations](images/nat-translations.png)

## Troubleshooting

Real problems found and fixed during the build, with symptoms, diagnosis and fix:
[docs/troubleshooting.md](docs/troubleshooting.md)

## Repository structure

```
├── README.md
├── packet-tracer/     small-office-network.pkt
├── configs/           running-configs of R1-EDGE, ISP, SW1, SW2 (passwords removed)
├── docs/
│   ├── ip-addressing-plan.md
│   ├── verification.md
│   └── troubleshooting.md
└── images/            topology and test screenshots
```

## How to reproduce

1. Open `packet-tracer/small-office-network.pkt` in Cisco Packet Tracer 8.x, **or**
2. Build the topology from the [cabling table](docs/ip-addressing-plan.md#cabling) and paste the files from `configs/` into each device.
3. Set PCs to DHCP, the printer and PC-ADMIN to their static IPs, and enable DNS/HTTP on SRV-INTERNET.
4. Run the tests in [docs/verification.md](docs/verification.md).

> Lab credentials have been removed from the configs. Set your own before use.

## Possible improvements

- DHCP snooping and Dynamic ARP Inspection
- Redundant uplinks with EtherChannel and HSRP
- Syslog / NTP / SNMP monitoring
- IPv6 dual-stack
- A stateful firewall instead of stateless ACLs

## Author

**Stergios** – IT / Networking (CCNA level)
[LinkedIn](https://www.linkedin.com/in/YOUR-PROFILE) · [GitHub](https://github.com/YOUR-USERNAME)
