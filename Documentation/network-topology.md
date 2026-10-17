# Network Topology and Architecture

## 1. Project Overview

This project simulates an enterprise network connecting Cairo HQ and Alexandria Branch using GNS3, Cisco IOS devices, FortiGate firewalls, Windows Server, and VMware Workstation.

The architecture includes:

- Redundant core switching at Cairo HQ.
- FortiGate Active-Passive High Availability.
- Dual ISP connectivity.
- VLAN segmentation and HSRP.
- Multi-area OSPF routing.
- Site-to-site IPsec VPN tunnels.
- A DMZ for a web server.
- Windows Active Directory and DNS services.
- A legacy RIP routing domain connected through ALEX-ASBR.
- VMware VMnet8 for Internet connectivity.

This document describes the recorded design. Features that have not been verified are identified as planned or requiring verification.

## 2. Cairo HQ

### 2.1 Core Switching

The Cairo HQ core layer consists of:

- `HQ-CORE-01`
- `HQ-CORE-02`

The design uses VLANs, 802.1Q trunking, EtherChannel, spanning tree, HSRP, Layer 3 SVIs, DHCP relay, and OSPF.

The Cairo VLANs are documented in [`ip-addressing-vlans.md`](ip-addressing-vlans.md).

### 2.2 Core-to-FortiGate Transit

| VLAN | Name | Network | Core Address | FortiGate Address |
|---:|---|---|---|---|
| 901 | FW-TRANSIT-A | `192.168.0.128/30` | CORE-1: `192.168.0.129` | `192.168.0.130` |
| 902 | FW-TRANSIT-B | `192.168.0.136/30` | CORE-2: `192.168.0.137` | `192.168.0.138` |

The supplied CORE-1 configuration confirms VLAN 901 at `192.168.0.129/30` and a default route through `192.168.0.130`.

Verify the current CORE-2 configuration and physical interface mappings against the running GNS3 topology.

### 2.3 FortiGate High Availability

The Cairo firewall pair consists of:

- `FG-HQ-01`
- `FG-HQ-02`

The intended HA mode is Active-Passive.

| Function | Interface |
|---|---|
| WAN1 | `port1` |
| WAN2 | `port2` |
| HA heartbeat | `port3`, `port4` |
| Core connections | `port5`, `port6` |
| DMZ | `port7` |
| Dedicated management | `port8` |

The recorded monitored interfaces are `port1`, `port2`, `port5`, and `port6`.

Verify HA synchronization, cluster health, and failover behavior before describing redundancy as successfully tested. The collected role-change and recovery evidence does not independently prove uninterrupted traffic during failover.

### 2.4 Dual ISP Connectivity

The simulated WAN addressing is:

| Component | Address |
|---|---|
| FortiGate WAN1 | `198.51.100.2/29` |
| ISP-A gateway | `198.51.100.1` |
| WAN1 VIP external address | `198.51.100.2` |
| FortiGate WAN2 | `203.0.113.2/29` |
| ISP-B gateway | `203.0.113.1` |
| WAN2 VIP external address | `203.0.113.2` |

The external VIP addresses above reflect the recorded VIP configuration; they use the same addresses as their respective FortiGate WAN interfaces.

ISP-A connects to the INET node over `198.51.100.248/30`:

- ISP-A Fa0/0: `198.51.100.249`
- INET Fa0/0: `198.51.100.250`

ISP-B connects to the INET node over `203.0.113.248/30`:

- ISP-B Fa0/0: `203.0.113.249`
- INET Fa1/0: `203.0.113.250`

VMware VMnet8 represents the lab's Internet connection. No additional internal subnet or host configuration is assumed for VMnet8.

WAN failover must be assessed using routing tables, link-monitor status, and end-to-end connectivity tests. The existing verification record documents observed failover and recovery behavior, but also records limitations in the WAN1 monitor results.

### 2.5 DMZ

The recorded DMZ subnet is `192.168.0.144/28`.

| Component | Address |
|---|---|
| FortiGate DMZ interface | `192.168.0.145` |
| WEB01 | `192.168.0.146` |

The recorded VIP configurations map external TCP port 443 to `192.168.0.146:443` through both WAN interfaces.

External HTTPS access through WAN1 and WAN2 has not been fully validated. The WAN2 test recorded a reverse-path check failure and packet drop.

## 3. Alexandria Branch

### 3.1 ALEX-CORE

The supplied configuration records the following addresses:

| Interface | Address |
|---|---|
| Ethernet0/0 | `10.10.1.1/30` |
| Ethernet0/1 | `10.10.1.5/30` |
| Loopback0 | `10.10.0.177/32` |
| VLAN 10 SVI | `10.10.0.2/25` |
| VLAN 20 SVI | `10.10.0.130/27` |
| VLAN 30 SVI | `10.10.0.193/28` |
| VLAN 99 SVI | `10.10.0.162/28` |

The branch VLANs are USERS, SERVERS, SALES, and MANAGEMENT.

The supplied interface output showed these four SVIs as down/down. Their current operational state must be verified.

### 3.2 OSPF Configuration

The supplied ALEX-CORE configuration records:

- OSPF process: `1`
- Router ID: `5.5.5.5`
- Area 0 network: `10.10.1.0/30`
- Area 2 network: `10.10.1.4/30`
- Area 2 summary: `10.10.0.0/24`
- Default route via `10.10.1.2`

The intended link addressing is:

- FG-ALEX to ALEX-CORE: `10.10.1.0/30`
- ALEX-CORE to ALEX-ASBR: `10.10.1.4/30`

The supplied ALEX-CORE configuration assigns `10.10.1.1/30` to Ethernet0/0, while the design handoff identifies that address as FG-ALEX's address. Confirm the actual peer configuration and cabling before making changes.

### 3.3 Legacy Routing

The legacy routing domain is planned to use RIPv2 with ALEX-ASBR as the redistribution point between RIP and OSPF.

The recorded design includes:

- `10.99.0.0/22` as the planned address space.
- `10.99.0.0/30` and `10.99.0.4/30` as point-to-point networks.
- `10.99.1.0/27` and `10.99.2.0/27` as legacy LANs.

The routing table on HQ-CORE-01 displayed `10.99.0.0/22` as an OSPF External Type 2 route. This confirms that the external route existed in the routing table, but does not establish the source protocol or prove the complete redistribution configuration.

Verify ALEX-ASBR's current configuration and routing behavior before documenting redistribution as operational.

## 4. Cairo–Alexandria VPN

The design includes two route-based IPsec tunnels:

| Tunnel | Network | Cairo Address | Alexandria Address | OSPF Cost |
|---|---|---|---|---:|
| Tunnel A — Primary | `172.16.0.0/30` | `172.16.0.1` | `172.16.0.2` | 10 |
| Tunnel B — Backup | `172.16.0.4/30` | `172.16.0.5` | `172.16.0.6` | 100 |

Tunnel A is intended to be the preferred path. Tunnel B is intended to provide an alternative path.

Collected FortiGate outputs showed Security Association activity and packet counters for both configured tunnels, as well as OSPF neighbor relationships over the displayed tunnel interfaces on FG-ALEX-01.

These observations confirm tunnel and routing activity at the time of capture. End-to-end application connectivity, automatic failover, and recovery to the preferred path remain unverified.

The remote-access VPN pool `172.16.1.0/27` is reserved in the design; its current configuration is unverified.

## 5. Routing Architecture

The intended routing design is:

- Cairo internal VLAN networks: OSPF Area 1.
- Cairo core-to-firewall transit: OSPF Area 0.
- Alexandria local VLAN networks: OSPF Area 2.
- Alexandria backbone link: OSPF Area 0.
- Planned Cairo summary: `192.168.0.0/25`.
- Planned Alexandria summary: `10.10.0.0/24`.
- ALEX-ASBR: planned RIP/OSPF redistribution point.
- Tunnel A: OSPF cost 10.
- Tunnel B: OSPF cost 100.

The project design specifies that the FortiGates should not originate OSPF default routes, leaving ALEX-ASBR as the planned redistribution point.

Collected routing evidence showed OSPF adjacencies and an OSPF External Type 2 route for `10.99.0.0/22` on HQ-CORE-01. The exact redistribution source and configuration still require confirmation.

Verify OSPF neighbors, route learning, summary propagation, default routes, and redistribution using live device outputs.

## 6. Windows Server Services

The recorded Windows Server addressing is:

| Server | Address | Gateway | Notes |
|---|---|---|---|
| DC01 | `192.168.0.36/28` | `192.168.0.33` | Domain: `test.local` |
| DC02 | `192.168.0.37/28` | `192.168.0.33` | DNS recorded as `192.168.0.36` |

The project includes Active Directory Domain Services, DNS, and DHCP.

Verify domain-controller replication, DNS resolution, DHCP scopes, and client connectivity before describing these services as fully operational.

## 7. Management Network

The updated management plan uses:

- Network: `192.168.22.0/24`
- Recorded addresses: `192.168.22.100`, `192.168.22.110`, and `192.168.22.120`

The device mapping, gateway, and VLAN/interface assignments for these addresses have not been confirmed.

## 8. Internet Connectivity

VMware VMnet8 represents the lab's Internet/NAT connection.

The intended Internet path depends on correct routing through the simulated ISP and firewall topology, together with the required firewall policies.

VMnet8 is treated as the external Internet connection for the lab; no additional internal subnet is assumed.

## 9. Verification Checklist

- [ ] Verify HQ-CORE-01 and HQ-CORE-02 transit interfaces and physical mappings.
- [ ] Verify FortiGate HA health and synchronization.
- [ ] Verify controlled HA failover and traffic continuity.
- [ ] Confirm DMZ interface assignments and WEB01 connectivity.
- [ ] Complete WAN1/WAN2 routing and failover validation.
- [ ] Reconcile FG-ALEX and ALEX-CORE link addressing.
- [ ] Verify Alexandria SVI operational status.
- [ ] Verify OSPF neighbors, learned routes, summaries, and default routes.
- [ ] Verify both IPsec tunnels and route preference.
- [ ] Test end-to-end connectivity between Cairo and Alexandria.
- [ ] Verify ALEX-ASBR and RIP/OSPF redistribution.
- [ ] Confirm device assignments for the management addresses.
- [ ] Verify DC01/DC02 replication, DNS, and DHCP.

---

**Documentation status:** This file describes the current design based on recorded project information. Items marked planned or requiring verification must not be presented as successfully tested until supporting evidence has been collected.
