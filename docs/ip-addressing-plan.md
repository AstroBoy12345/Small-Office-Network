# IP Addressing Plan

## VLANs and subnets

| VLAN | Name | Subnet | Gateway (R1-EDGE) | DHCP range | Purpose |
|---|---|---|---|---|---|
| 10 | STAFF | 192.168.10.0/24 | 192.168.10.1 (G0/1.10) | .21 – .254 | Employee PCs, printer |
| 20 | VOICE | 192.168.20.0/24 | 192.168.20.1 (G0/1.20) | .21 – .254 | IP phones (option 150) |
| 30 | GUEST | 192.168.30.0/24 | 192.168.30.1 (G0/1.30) | .21 – .254 | Visitors – internet only |
| 99 | MGMT | 192.168.99.0/24 | 192.168.99.1 (G0/1.99) | none (static) | Switch SVIs, admin PC |
| 666 | BLACKHOLE | – | – | – | Unused ports (not on any trunk) |
| 999 | NATIVE | – | – | – | Trunk native VLAN, no users |

Addresses .1 – .20 in every DHCP subnet are excluded and reserved for gateways and static devices.

## Point-to-point and external networks

| Network | Device | Interface | Address |
|---|---|---|---|
| 203.0.113.0/30 (WAN) | R1-EDGE | G0/0 | 203.0.113.2 (NAT outside) |
| | ISP | G0/0 | 203.0.113.1 |
| 198.51.100.0/24 (internet LAN) | ISP | G0/1 | 198.51.100.1 |
| | SRV-INTERNET | Fa0 | 198.51.100.10 (DNS + HTTP) |
| Simulated public host | ISP | Loopback0 | 8.8.8.8/32 |

`203.0.113.0/24` and `198.51.100.0/24` are reserved documentation ranges (RFC 5737).

## Static addresses

| Device | IP | Gateway |
|---|---|---|
| SW1 (Vlan99) | 192.168.99.11/24 | 192.168.99.1 |
| SW2 (Vlan99) | 192.168.99.12/24 | 192.168.99.1 |
| PC-ADMIN | 192.168.99.100/24 | 192.168.99.1 |
| PRINTER | 192.168.10.5/24 | 192.168.10.1 |

## Switch port map

### SW1 (STP root, priority 4096)

| Port | Mode | VLAN | Connected to |
|---|---|---|---|
| Gi0/1 | Trunk | native 999 | R1-EDGE G0/1 |
| Gi0/2 | Trunk | native 999 | SW2 Gi0/1 |
| Fa0/1 – 12 | Access + voice | 10 / voice 20 | PC-STAFF1 (Fa0/1), PRINTER (Fa0/2) |
| Fa0/13 – 23 | Access, shutdown | 666 | unused |
| Fa0/24 | Access | 99 | PC-ADMIN |

### SW2 (backup root, priority 8192)

| Port | Mode | VLAN | Connected to |
|---|---|---|---|
| Gi0/1 | Trunk | native 999 | SW1 Gi0/2 |
| Gi0/2 | Access, shutdown | 666 | unused |
| Fa0/1 – 12 | Access + voice | 10 / voice 20 | PC-STAFF2 (Fa0/1) |
| Fa0/13 – 20 | Access | 30 | LAPTOP-GUEST (Fa0/13) |
| Fa0/21 – 24 | Access, shutdown | 666 | unused |

## Cabling

| From | To | Cable |
|---|---|---|
| R1-EDGE G0/0 | ISP G0/0 | Copper cross-over |
| R1-EDGE G0/1 | SW1 Gi0/1 | Copper straight-through |
| SW1 Gi0/2 | SW2 Gi0/1 | Copper cross-over |
| ISP G0/1 | SRV-INTERNET Fa0 | Copper straight-through |
| PC-STAFF1 | SW1 Fa0/1 | Copper straight-through |
| PRINTER | SW1 Fa0/2 | Copper straight-through |
| PC-ADMIN | SW1 Fa0/24 | Copper straight-through |
| PC-STAFF2 | SW2 Fa0/1 | Copper straight-through |
| LAPTOP-GUEST | SW2 Fa0/13 | Copper straight-through |
