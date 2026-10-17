# Network Topology and Architecture

## 1. Project Overview

This project simulates an enterprise network connecting Cairo HQ and Alexandria Branch using GNS3, Cisco IOS devices, FortiGate firewalls, Windows Server, and VMware Workstation.

The architecture includes:

- Redundant core switching at Cairo HQ.
- FortiGate Active-Passive High Availability.
- Dual ISP connectivity with automatic WAN failover and recovery.
- VLAN segmentation and HSRP.
- Multi-area OSPF routing.
- Two site-to-site IPsec VPN tunnels with WAN path failover.
- A DMZ hosting the web server.
- Windows Active Directory and DNS services.
- A legacy RIP routing domain connected through ALEX-ASBR.
- VMware VMnet8 for Internet connectivity.

This document describes the recorded architecture and the functionality demonstrated by the project's collected configuration outputs, tests, and screenshots.

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

The recorded FortiGate routing table also shows connected transit networks on `port5` and `port6`.

### 2.3 FortiGate High Availability

The Cairo firewall pair consists of:

- `FG-HQ-1` — Primary
- `FG-HQ-2` — Secondary

The cluster operates in Active-Passive mode.

| Function | Interface |
|---|---|
| WAN1 | `port1` |
| WAN2 | `port2` |
| HA heartbeat | `port3`, `port4` |
| Core connections | `port5`, `port6` |
| DMZ | `port7` |
| Dedicated management | `port8` |

The recorded monitored interfaces are `port1`, `port2`, `port5`, and `port6`.

Collected HA status output showed:

- HA health status: `OK`
- Configuration synchronization: `in-sync` on both members
- `FG-HQ-1` operating as Primary
- `FG-HQ-2` operating as Secondary
- Heartbeat interfaces `port3` and `port4` up

Additional screenshots document a role change to `FG-HQ-2` and subsequent recovery.

**Result:** HA cluster status, synchronization, and role-change/recovery behavior are documented. These observations are distinct from a controlled test proving uninterrupted application traffic during HA failover.

Evidence:

- `Images/FG-HQ-HA-Status-In-Sync.png`
- `Images/FG-HQ-HA-Status-Primary.png`
- `Images/FG-HQ-HA-Status-Secondary.png`
- `Images/FG-HQ-HA-Failover-to-HQ-2.png`
- `Images/FG-HQ-HA-Recovery.png`

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

The recorded VIP external addresses use the same IP addresses as their respective FortiGate WAN interfaces. This is retained as observed configuration and should be distinguished from any future addressing redesign.

ISP-A connects to the INET node over `198.51.100.248/30`:

- ISP-A Fa0/0: `198.51.100.249`
- INET Fa0/0: `198.51.100.250`

ISP-B connects to the INET node over `203.0.113.248/30`:

- ISP-B Fa0/0: `203.0.113.249`
- INET Fa1/0: `203.0.113.250`

VMware VMnet8 represents the lab's external Internet/NAT connection.

The recorded Dual-WAN tests demonstrated:

1. WAN1 selected as the preferred default route during normal operation.
2. WAN1 monitor transitioning to `dead` during the simulated ISP-A failure.
3. WAN2 monitor remaining `alive`.
4. Default-route failover from WAN1 to WAN2.
5. WAN1 monitor recovery and return of the preferred default route to WAN1.

**Result: PASS — Dual-WAN default-route failover and recovery were demonstrated in the recorded lab tests.**

The verification record also includes monitor output showing packet loss during one recovery observation. This does not negate the demonstrated route change, but should remain documented as a monitoring observation rather than being silently removed.

Evidence:

- `Images/Dual-WAN-HA-Before-Failover.png`
- `Images/Dual-WAN-Routing-Before-Failover.png`
- `Images/Dual-WAN-Monitor-Before-Failover.png`
- `Images/Dual-WAN-Monitor-During-Failover.png`
- `Images/Dual-WAN-Failover-to-WAN2.png`
- `Images/Dual-WAN-Ping-WAN2-Success.png`
- `Images/Dual-WAN-Recovery-to-WAN1.png`
- `Images/Dual-WAN-Ping-WAN1-Recovery.png`

### 2.5 DMZ

The recorded DMZ subnet is `192.168.0.144/28`.

| Component | Address |
|---|---|
| FortiGate DMZ interface | `192.168.0.145` |
| WEB01 | `192.168.0.146` |

The DMZ web server is reachable from the internal IT/server network using the configured internal firewall policy and the documented lab tests.

The internal DMZ access functionality is treated as completed project functionality.

The recorded VIP configurations map external TCP port 443 to `192.168.0.146:443` through both WAN interfaces.

The remaining issue is specifically **external HTTPS publishing through WAN2**. The recorded WAN2 debug flow showed destination NAT to `192.168.0.146:443`, followed by a reverse-path check failure and packet drop.

External HTTPS publishing through WAN1 and WAN2 must be distinguished from the already demonstrated internal LAN-to-DMZ functionality.

**Result:** LAN-to-DMZ access is documented as completed. WAN2-to-DMZ HTTPS remains the outstanding external publishing issue.

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

The supplied interface output showed these four SVIs as down/down at the time of capture. This is historical evidence; it should not be interpreted as their current status without a newer check.

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

The supplied ALEX-CORE configuration assigns `10.10.1.1/30` to Ethernet0/0, while the design handoff identifies that address as FG-ALEX's address. This addressing discrepancy should be reconciled against the current running configuration before any change is made.

### 3.3 Legacy Routing

The legacy routing domain uses the planned RIPv2/OSPF integration design with ALEX-ASBR as the redistribution point.

The recorded design includes:

- `10.99.0.0/22` as the planned address space.
- `10.99.0.0/30` and `10.99.0.4/30` as point-to-point networks.
- `10.99.1.0/27` and `10.99.2.0/27` as legacy LANs.

The routing table on HQ-CORE-01 displayed `10.99.0.0/22` as an OSPF External Type 2 route. This confirms that the external route existed in the routing table, while the exact source protocol and redistribution statements are separate configuration details.

## 4. Cairo–Alexandria VPN

The design includes two route-based IPsec tunnels:

| Tunnel | Network | Cairo Address | Alexandria Address | OSPF Cost |
|---|---|---|---|---:|
| Tunnel A — Primary | `172.16.0.0/30` | `172.16.0.1` | `172.16.0.2` | 10 |
| Tunnel B — Backup | `172.16.0.4/30` | `172.16.0.5` | `172.16.0.6` | 100 |

Tunnel A is intended to be the preferred path. Tunnel B provides the alternate path.

The collected evidence documents:

- Both IPsec tunnels established before failover.
- Incoming and outgoing tunnel data counters.
- WAN1 selected as the preferred route before failover.
- WAN1 unavailable while WAN2 remained operational during the recorded test.
- Routing and VPN state switching to the WAN2 path.
- Recovery of the preferred route to WAN1.

**Result: PASS — VPN path failover to WAN2 and recovery to WAN1 were demonstrated by the recorded lab evidence.**

Evidence:

- `Images/VPN-Tunnels-Before-Failover.png`
- `Images/VPN-Routing-Before-Failover.png`
- `Images/VPN-Routing-WAN1-Primary.png`
- `Images/VPN-WAN1-Down-WAN2-Up.png`
- `Images/VPN-Failover-WAN1-Down-WAN2-Up.png`
- `Images/VPN-Failover-to-WAN2.png`
- `Images/VPN-Recovery-to-WAN1.png`

The remote-access VPN pool `172.16.1.0/27` is reserved in the design; its current configuration is separate from the site-to-site VPN tests above.

## 5. Routing Architecture

The routing design includes:

- Cairo internal VLAN networks using OSPF Area 1.
- Cairo core-to-firewall transit using OSPF Area 0.
- Alexandria local VLAN networks using OSPF Area 2.
- Alexandria backbone connectivity using OSPF Area 0.
- Cairo summary: `192.168.0.0/25`.
- Alexandria summary: `10.10.0.0/24`.
- ALEX-ASBR as the RIP/OSPF redistribution point.
- Tunnel A with OSPF cost 10.
- Tunnel B with OSPF cost 100.

Collected routing evidence showed OSPF adjacencies, a static default route toward the FortiGate transit connection, and an OSPF External Type 2 route for `10.99.0.0/22` on HQ-CORE-01.

The route tables also showed the remote networks through the WAN1-associated and WAN2-associated VPN interfaces in their respective recorded states.

The exact redistribution source and statements are documented separately from the observed route-table entries.

## 6. Windows Server Services

The recorded Windows Server addressing is:

| Server | Address | Gateway | Notes |
|---|---|---|---|
| DC01 | `192.168.0.36/28` | `192.168.0.33` | Domain: `test.local` |
| DC02 | `192.168.0.37/28` | `192.168.0.33` | DNS recorded as `192.168.0.36` |

The project includes Active Directory Domain Services, DNS, and DHCP.

The specific operational status of domain-controller replication, DNS resolution, DHCP scopes, and client connectivity should be described according to their respective lab test evidence.

## 7. Management Network

The updated management plan uses:

- Network: `192.168.22.0/24`
- Recorded addresses: `192.168.22.100`, `192.168.22.110`, and `192.168.22.120`

The device mapping, gateway, and VLAN/interface assignments for these addresses are maintained in the addressing documentation.

## 8. Internet Connectivity

VMware VMnet8 represents the lab's external Internet/NAT connection.

The Internet path depends on the simulated ISP links, FortiGate routing, link-monitor behavior, and the required firewall policies.

The recorded tests demonstrated default-route failover from WAN1 to WAN2 and recovery to WAN1.

The separate WAN2-to-DMZ HTTPS publishing issue is documented in the firewall policy verification record and should not be confused with Dual-WAN routing failover.

## 9. Verification Summary

| Feature | Recorded Result |
|---|---|
| FortiGate HA health and configuration synchronization | PASS — healthy and in-sync in captured output |
| HA role change and recovery | Observed and documented |
| Dual-WAN default-route failover | PASS |
| Dual-WAN preferred-route recovery | PASS |
| Site-to-site IPsec tunnel operation | PASS |
| VPN path failover to WAN2 | PASS |
| VPN recovery to WAN1 | PASS |
| LAN-to-DMZ access | Completed and documented |
| External HTTPS through WAN2 to DMZ | Outstanding issue — reverse-path check failure observed |
| OSPF adjacencies and recorded routes | Confirmed in captured routing outputs |
| RIP/OSPF redistribution source configuration | Requires configuration-level confirmation |
| Controlled HA failover with application traffic continuity | Not established by role-change evidence alone |

## 10. Remaining Work

The remaining work is limited to the following documented items:

- Resolve and retest external HTTPS access through WAN2 to WEB01 in the DMZ.
- Investigate the observed WAN1 monitor packet-loss result after recovery if it recurs.
- Verify the current ALEX-CORE interface addressing against the intended FG-ALEX/ALEX-ASBR link design.
- Confirm the exact source protocol and configuration statements for RIP/OSPF redistribution.
- Investigate the origin and purpose of the recorded `172.16.0.x/32` routes.
- If full HA service continuity is a project requirement, capture a controlled HA failover test that verifies actual application traffic.

## 11. Conclusion

The lab documents a multi-site enterprise network with redundant Cairo core switching, FortiGate Active-Passive HA, dual ISP paths, multi-area OSPF, two site-to-site IPsec tunnels, a segmented DMZ, and Windows Server services.

The recorded evidence demonstrates Dual-WAN route failover and recovery, VPN path failover and recovery, active VPN tunnel operation, OSPF routing activity, and internal LAN-to-DMZ access.

The outstanding external publishing issue is WAN2-to-DMZ HTTPS. It is not a general failure of the VPN, Dual-WAN routing, or internal DMZ access.

**Documentation status:** The completed and demonstrated functions are recorded as such. Remaining items are limited to the specific issues and configuration details listed above.
