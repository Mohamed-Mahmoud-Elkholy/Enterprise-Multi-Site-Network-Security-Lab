# Enterprise Multi-Site Network & Security Infrastructure Lab
## IP Addressing, VLANs, Routing & Infrastructure

**Environment:** GNS3, Cisco IOS, FortiGate, Windows Server, VMware Workstation  
**Sites:** Cairo HQ and Alexandria Branch

> **Documentation policy:** This document distinguishes configured values, design values, and items requiring verification. An IP address in a plan does not by itself prove that a link or service is operational.

## 1. Cairo HQ — VLAN and VLSM Plan

| VLAN ID | Name | Network | Subnet Mask | HSRP Virtual IP |
|---:|---|---|---|---|
| 10 | USERS | 192.168.0.0/27 | 255.255.255.224 | 192.168.0.1 |
| 20 | SERVERS | 192.168.0.32/28 | 255.255.255.240 | 192.168.0.33 |
| 30 | IT | 192.168.0.48/28 | 255.255.255.240 | 192.168.0.49 |
| 40 | HR | 192.168.0.64/28 | 255.255.255.240 | 192.168.0.65 |
| 50 | FINANCE | 192.168.0.80/28 | 255.255.255.240 | 192.168.0.81 |
| 60 | SALES | 192.168.0.96/29 | 255.255.255.248 | 192.168.0.97 |
| 70 | GUEST | 192.168.0.104/29 | 255.255.255.248 | 192.168.0.105 |
| 99 | MANAGEMENT | 192.168.0.112/28 | 255.255.255.240 | 192.168.0.113 |

The Cairo VLAN networks are summarized as `192.168.0.0/25` in the routing design. The HSRP virtual IP is the intended client default gateway. DHCP scopes must exclude virtual IPs, actual SVI addresses, servers, and other statically assigned infrastructure addresses.

## 2. Cairo HQ — Core Switching

Documented switching features:
- VLANs 10, 20, 30, 40, 50, 60, 70, and 99
- 802.1Q trunking
- EtherChannel using LACP
- Rapid-PVST spanning tree
- HSRP gateway redundancy
- Layer 3 SVIs and DHCP relay

### HSRP Status Recorded for CORE-1

| VLAN | CORE-1 Priority | Recorded State | Virtual IP |
|---:|---:|---|---|
| 10 | 110 | Active | 192.168.0.1 |
| 20 | 110 | Active | 192.168.0.33 |
| 30 | 110 | Active | 192.168.0.49 |
| 40 | 90 | Standby | 192.168.0.65 |
| 50 | 110 | Active | 192.168.0.81 |
| 60 | 90 | Standby | 192.168.0.97 |
| 70 | 110 | Active | 192.168.0.105 |
| 99 | 90 | Standby | 192.168.0.113 |

These are recorded observations, not a guarantee of the current state. Verify both core switches before documenting current HSRP roles.

## 3. Cairo HQ — Core and Firewall Transit

The current design retains two Core-to-FortiGate transit networks. The previously listed networks `192.168.0.132/30` and `192.168.0.140/30` have been removed.

| Link | Network | Core Address | FortiGate Address |
|---|---|---|---|
| CORE-1 ↔ FG-HQ | 192.168.0.128/30 | 192.168.0.129 | 192.168.0.130 |
| CORE-2 ↔ FG-HQ | 192.168.0.136/30 | 192.168.0.137 | 192.168.0.138 |

### Firewall Transit VLANs

| VLAN ID | Name | Core Switch |
|---:|---|---|
| 901 | FW-TRANSIT-A | CORE-1 |
| 902 | FW-TRANSIT-B | CORE-2 |

The supplied CORE-1 configuration contains SVI VLAN 901 at `192.168.0.129/30` and a default route via `192.168.0.130`. Verify the actual FortiGate interface configuration and physical connections against the current lab. Do not restore the removed transit networks from older plans.

## 4. Cairo HQ — FortiGate HA

### HA Design
- Devices: `FG-HQ-01` and `FG-HQ-02`
- HA mode: Active-Passive
- Heartbeat interfaces: ports 3 and 4
- Previously recorded monitored interfaces: ports 1, 2, 5, and 6
- WAN interfaces: ports 1 and 2
- Core-facing interfaces: ports 5 and 6
- DMZ interface: port 7

The project handoff records Cairo FortiGate HA as configured. Current HA health and failover behavior still require live verification.

### Firewall Loopbacks

| Device | Loopback Address |
|---|---|
| FG-HQ-01 | 192.168.1.9/32 |
| FG-HQ-02 | 192.168.1.10/32 |

The addressing plan places these addresses within `192.168.1.8/29`. Verify the cluster router-ID and loopback behavior before treating the units as independent routing identities.

## 5. Cairo HQ — WAN, ISP and Internet

### FortiGate WAN Interfaces

| Interface | Network | FortiGate Address | ISP Gateway | Planned VIP |
|---|---|---|---|---|
| WAN1 / port1 | 198.51.100.0/29 | 198.51.100.2/29 | 198.51.100.1 | 198.51.100.3 |
| WAN2 / port2 | 203.0.113.0/29 | 203.0.113.2/29 | 203.0.113.1 | 203.0.113.3 |

VIPs are design entries and must not be treated as operational until the virtual IP objects and related policies are verified.

### ISP-A to INET

| Device / Interface | Address |
|---|---|
| Network | 198.51.100.248/30 |
| ISP-A Fa0/0 | 198.51.100.249 |
| INET Fa0/0 | 198.51.100.250 |

### ISP-B to INET

| Device / Interface | Address |
|---|---|
| Network | 203.0.113.248/30 |
| ISP-B Fa0/0 | 203.0.113.249 |
| INET Fa1/0 | 203.0.113.250 |

### WAN Monitoring

The handoff records a WAN1 link monitor targeting `198.51.100.250`, using gateway `198.51.100.1` and source address `198.51.100.2`. The latest recorded status was alive with 0% packet loss.

A previously observed default-route selection used WAN2. Verify final route preference and failover against the current routing table. A link monitor and an SD-WAN Performance SLA are different features.

## 6. DMZ and Public Web Service

| Item | Network / Address | Status |
|---|---|---|
| DMZ subnet | 192.168.0.144/28 | Addressing plan |
| FortiGate DMZ address | 192.168.0.145 | Recorded in plan |
| WEB01 | 192.168.0.146 | Planned; verify before use |

The DMZ is intended to host a web server reachable through controlled firewall policies and public VIP/NAT configuration.

**Address conflict warning:** An earlier validation note also assigned `192.168.0.145` to CORE-1 and `192.168.0.146` to an `INSIDE-A` interface. Check the actual topology and current interface configurations before using these addresses.

## 7. Alexandria Branch — VLAN and IP Addressing

| VLAN ID | Name | Network | Subnet Mask | ALEX-CORE SVI |
|---:|---|---|---|---|
| 10 | USERS | 10.10.0.0/25 | 255.255.255.128 | 10.10.0.2 |
| 20 | SERVERS | 10.10.0.128/27 | 255.255.255.224 | 10.10.0.130 |
| 30 | SALES | 10.10.0.192/28 | 255.255.255.240 | 10.10.0.193 |
| 99 | MANAGEMENT | 10.10.0.160/28 | 255.255.255.240 | 10.10.0.162 |

The Alexandria VLAN plan follows the supplied ALEX-CORE running configuration and VLAN table.

### Alexandria Core Loopbacks and Routed Links

| Interface / Link | Address | Network | Status |
|---|---|---|---|
| ALEX-CORE Loopback0 | 10.10.0.177/32 | 10.10.0.176/29 allocation | Present in running configuration |
| FG-ALEX ↔ ALEX-CORE | Planned FG `.1`, Core `.2` | 10.10.1.0/30 | Address conflict; verify |
| ALEX-CORE ↔ ALEX-ASBR | Planned Core `.5`, ASBR `.6` | 10.10.1.4/30 | Core address present; peer unverified |
| ALEX-ASBR Loopback | 10.10.0.178/32 | 10.10.0.176/29 allocation | Design value; verify |

**Live-config discrepancy:** The supplied ALEX-CORE running configuration shows Ethernet0/0 at `10.10.1.1/30` and Ethernet0/1 at `10.10.1.5/30`. The handoff instead proposes ALEX-CORE `.2` on the first link and `.5` on the second. Confirm actual cable endpoints and peer configurations before changing anything.

The supplied ALEX-CORE output shows VLAN 10, 20, 30, and 99 SVIs as `down/down`. The VLANs exist, but operational Layer 3 connectivity was not demonstrated by that output.

## 8. Alexandria Branch — OSPF

The supplied ALEX-CORE configuration contains:
- OSPF process: 1
- Router ID: `5.5.5.5`
- Local area: Area 2
- Ethernet0/0 network `10.10.1.0/30` assigned to Area 0
- Ethernet0/1 network `10.10.1.4/30` assigned to Area 2
- Area 2 summary configured as `10.10.0.0/24`
- Default route configured through `10.10.1.2`

The planned Alexandria routing design includes ALEX-CORE as an Area Border Router and ALEX-ASBR as the redistribution point for the legacy RIP domain. Verify OSPF neighbors, learned routes, summary advertisement, and default-route reachability before claiming the design is operational.

## 9. Cairo–Alexandria Site-to-Site VPN

| Tunnel | Network | Cairo Address | Alexandria Address | Planned Cost |
|---|---|---|---|---:|
| Tunnel A — Primary | 172.16.0.0/30 | 172.16.0.1 | 172.16.0.2 | 10 |
| Tunnel B — Backup | 172.16.0.4/30 | 172.16.0.5 | 172.16.0.6 | 100 |

Remote-access VPN address pool reserved in the design: `172.16.1.0/27`.

The site-to-site tunnels and remote-access pool are design entries. Verify tunnel interfaces, IPsec status, routing, and failover before marking them completed.

## 10. Legacy RIP Domain

| Segment | Network | Purpose |
|---|---|---|
| Aggregate | 10.99.0.0/22 | Legacy routing domain |
| Link 1 | 10.99.0.0/30 | Point-to-point |
| Link 2 | 10.99.0.4/30 | Point-to-point |
| Legacy LAN 1 | 10.99.1.0/27 | Legacy LAN |
| Legacy LAN 2 | 10.99.2.0/27 | Legacy LAN |

The design places RIP-to-OSPF and OSPF-to-RIP redistribution on ALEX-ASBR only. Route tagging and filtering are intended to prevent routing loops. Peer addressing and actual redistribution configuration remain to be verified.

## 11. Management Network — Updated

The previous management network `192.168.1.16/28` and its `.17`–`.20` assignments have been removed from the current plan.

| Item | Value |
|---|---|
| Network | 192.168.22.0/24 |
| Subnet Mask | 255.255.255.0 |
| Address 1 | 192.168.22.100 |
| Address 2 | 192.168.22.110 |
| Address 3 | 192.168.22.120 |

The device-to-address mapping, gateway, VLAN, and interface assignments have not been specified. Do not assign these addresses to particular devices without confirmation.

## 12. Internet Access and VMware VMnet8

**VMnet8 represents the lab's Internet connection.** It is the external Internet/NAT network used by the lab environment. No additional INET-HOST device or separate `192.0.2.0/28` host LAN was added.

- VMware network: `VMnet8`
- Role in this lab: Internet connectivity for devices that are routed and permitted to access the Internet
- No separate INET-HOST node or LAN is part of the current design
- No additional VMnet8 subnet or host IP addresses are specified in this document

The ISP-A and ISP-B links documented in Section 5 are the lab's simulated dual-WAN path toward the Internet network represented by VMnet8.

## 13. Windows Server and DHCP

The handoff records:

| Server | Address | Notes |
|---|---|---|
| DC01 | 192.168.0.36/28 | Gateway 192.168.0.33; `test.local` domain |
| DC02 | 192.168.0.37/28 | Gateway 192.168.0.33; DNS recorded as 192.168.0.36 |

The project includes Windows Server, Active Directory, DNS, and planned DHCP scopes and relay configuration.

DHCP scopes must respect each VLAN subnet and exclude gateways, server addresses, SVI addresses, and other static assignments. DHCP failover, directory replication, and DNS health must be verified separately.

## 14. Routing and Redundancy Design

The intended routing architecture includes:
- Cairo core VLAN routing in OSPF Area 1
- Cairo core-to-firewall transit links in Area 0
- Alexandria local routing in Area 2
- Alexandria firewall-facing link in Area 0
- Route-based IPsec Tunnel A and Tunnel B between sites
- Cairo OSPF summary: `192.168.0.0/25`
- Alexandria OSPF summary: `10.10.0.0/24`
- HSRP for redundant client default gateways
- FortiGate Active-Passive HA in Cairo
- Dual ISP connectivity through ISP-A and ISP-B
- Internet access represented by VMware `VMnet8`

This section describes the intended design. Actual adjacency, route propagation, HA failover, and dual-WAN failover require verification evidence.

## 15. Removed or Superseded Entries

The following items are not part of the current design:
- Old management network `192.168.1.16/28` and addresses `.17`–`.20`
- Transit network `192.168.0.132/30`
- Transit network `192.168.0.140/30`
- Separate `INET-HOST LAN` `192.0.2.0/28`
- A separate INET-HOST node
- Any assumption that the new management addresses belong to specific devices without confirmation
- Any claim that the site-to-site tunnels, WEB01, public VIPs, or remote-access VPN are operational without verification

## 16. Verification Checklist

- [ ] Confirm CORE-1 and CORE-2 transit interfaces and addresses.
- [ ] Resolve the DMZ `.145` and `.146` addressing conflict.
- [ ] Confirm FortiGate HA status and monitored interfaces.
- [ ] Verify WAN1 and WAN2 routing and failover.
- [ ] Verify ALEX-CORE link addresses against the actual connected devices.
- [ ] Verify Alexandria SVIs become operational.
- [ ] Verify OSPF neighbors, routes, and summaries.
- [ ] Verify both IPsec tunnels and route preference.
- [ ] Verify RIP redistribution and loop prevention.
- [ ] Confirm implementation of the updated management network `192.168.22.0/24`.
- [ ] Verify Internet access through VMware VMnet8.
- [ ] Verify Windows Server, AD DS, DNS, and DHCP functionality.

---

**Project status:** This document consolidates the supplied addressing plan, running configurations, and user-confirmed removals/additions. Values explicitly marked for verification must remain unverified until checked in the live lab.
