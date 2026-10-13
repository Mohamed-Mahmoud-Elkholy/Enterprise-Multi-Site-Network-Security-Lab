# Enterprise Multi-Site Network & Security Infrastructure Lab
## IP Addressing, VLANs, Routing & Infrastructure

**Environment:** GNS3, Cisco IOS, FortiGate, Windows Server, VMware Workstation  
**Sites:** Cairo HQ and Alexandria Branch

> **Documentation policy:** This document distinguishes configured values, design values, and items requiring verification. An IP address in the plan does not by itself prove that a link or service is operational.

---

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

The Cairo VLAN networks are summarized as `192.168.0.0/25` in the routing design.

**Important:** The HSRP virtual IP is the default gateway for clients. DHCP scopes must exclude the virtual IP, actual SVI addresses, servers, and other statically assigned infrastructure addresses.

## 2. Cairo HQ — Core Switching

### VLANs and Switching

The documented switching features include:

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

## 3. Cairo HQ — Core-to-FortiGate Transit Links

The following addressing is recorded in the project handoff:

| Link | Network | Core Address | FortiGate Address |
|---|---|---|---|
| CORE-1 ↔ FG-HQ-01 | 192.168.0.128/30 | 192.168.0.129 | 192.168.0.130 |
| CORE-1 ↔ FG-HQ-02 | 192.168.0.132/30 | 192.168.0.133 | 192.168.0.134 |
| CORE-2 ↔ FG-HQ-01 | 192.168.0.136/30 | 192.168.0.137 | 192.168.0.138 |
| CORE-2 ↔ FG-HQ-02 | 192.168.0.140/30 | 192.168.0.141 | 192.168.0.142 |

The transit VLAN names recorded on the core switches are:

- VLAN 901 — `FW-TRANSIT-A` on CORE-1
- VLAN 902 — `FW-TRANSIT-B` on CORE-2

**Verification required:** The transit table above describes the planned addressing from the handoff. The supplied CORE-1 running configuration contains SVI VLAN 901 at `192.168.0.129/30` and a default route via `192.168.0.130`. The exact physical port-to-firewall mapping and the corresponding CORE-2 configuration must be checked against the current lab.

## 4. Cairo HQ — FortiGate HA

### HA Design

- Devices: `FG-HQ-01` and `FG-HQ-02`
- HA mode: Active-Passive
- Heartbeat interfaces: ports 3 and 4
- Previously recorded monitored interfaces: ports 1, 2, 5, and 6
- WAN interfaces: ports 1 and 2
- Core-facing interfaces: ports 5 and 6
- DMZ interface: port 7

The project handoff records the Cairo FortiGate HA configuration as completed. Current HA health and failover behavior still require live verification.

### Firewall Loopbacks

| Device | Loopback Address |
|---|---|
| FG-HQ-01 | 192.168.1.9/32 |
| FG-HQ-02 | 192.168.1.10/32 |

The addressing plan places these addresses within `192.168.1.8/29`. The correct OSPF router-ID and loopback behavior for an HA cluster must be verified before using these as independent routing identities.

## 5. Cairo HQ — WAN, ISP and Internet

### FortiGate WAN Interfaces

| Interface | Network | FortiGate Address | ISP Gateway | Planned VIP |
|---|---|---|---|---|
| WAN1 / port1 | 198.51.100.0/29 | 198.51.100.2/29 | 198.51.100.1 | 198.51.100.3 |
| WAN2 / port2 | 203.0.113.0/29 | 203.0.113.2/29 | 203.0.113.1 | 203.0.113.3 |

The VIP addresses are design entries and must not be treated as operational until the virtual IP objects and associated policies are verified.

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

The project also contains a previously observed default-route selection through WAN2. The final route preference and failover behavior must be verified against the current routing table. A link monitor is not the same as an SD-WAN Performance SLA.

## 6. DMZ and Public Web Service

| Item | Network / Address | Status |
|---|---|---|
| DMZ subnet | 192.168.0.144/28 | Addressing plan |
| FortiGate DMZ address | 192.168.0.145 | Recorded in the plan |
| WEB01 | 192.168.0.146 | Planned; verify before use |

The DMZ is intended to host a web server reachable through controlled firewall policies and public VIP/NAT configuration.

**Address conflict warning:** An earlier validation note also assigned `192.168.0.145` to CORE-1 and `192.168.0.146` to an `INSIDE-A` interface. Do not configure these conflicting assignments without checking the actual topology and current interface configurations.

## 7. Alexandria Branch — VLAN and IP Addressing

| VLAN ID | Name | Network | Subnet Mask | ALEX-CORE SVI |
|---:|---|---|---|---|
| 10 | USERS | 10.10.0.0/25 | 255.255.255.128 | 10.10.0.2 |
| 20 | SERVERS | 10.10.0.128/27 | 255.255.255.224 | 10.10.0.130 |
| 30 | SALES | 10.10.0.192/28 | 255.255.255.240 | 10.10.0.193 |
| 99 | MANAGEMENT | 10.10.0.160/28 | 255.255.255.240 | 10.10.0.162 |

The Alexandria VLAN plan above follows the supplied ALEX-CORE running configuration and VLAN table.

### Alexandria Core Loopbacks and Routed Links

| Interface / Link | Address | Network | Status |
|---|---|---|---|
| ALEX-CORE Loopback0 | 10.10.0.177/32 | 10.10.0.176/29 allocation | Present in running configuration |
| FG-ALEX ↔ ALEX-CORE | Planned FG `.1`, Core `.2` | 10.10.1.0/30 | Address conflict; verify |
| ALEX-CORE ↔ ALEX-ASBR | Planned Core `.5`, ASBR `.6` | 10.10.1.4/30 | Core address present; peer unverified |
| ALEX-ASBR Loopback | 10.10.0.178/32 | 10.10.0.176/29 allocation | Design value; verify |

**Important live-config discrepancy:** The supplied ALEX-CORE running configuration shows Ethernet0/0 at `10.10.1.1/30` and Ethernet0/1 at `10.10.1.5/30`. The handoff instead proposes ALEX-CORE `.2` on the first link and `.5` on the second. Confirm the actual cable endpoints and peer configurations before changing anything.

The supplied ALEX-CORE output also shows VLAN 10, 20, 30, and 99 SVIs as `down/down`. The VLANs exist, but operational Layer 3 connectivity has not been demonstrated by that output.

## 8. Alexandria Branch — OSPF

The supplied ALEX-CORE configuration contains:

- OSPF process: 1
- Router ID: `5.5.5.5`
- Local area: Area 2
- Ethernet0/0 network `10.10.1.0/30` assigned to Area 0
- Ethernet0/1 network `10.10.1.4/30` assigned to Area 2
- Area 2 summary configured as `10.10.0.0/24`
- Default route configured through `10.10.1.2`

The planned Alexandria routing design includes ALEX-CORE as an Area Border Router and ALEX-ASBR as the redistribution point for the legacy RIP domain.

OSPF neighbors, learned routes, summary advertisement, and default-route reachability must be verified before claiming the design is operational.

## 9. Cairo–Alexandria Site-to-Site VPN

| Tunnel | Network | Cairo Address | Alexandria Address | Planned Cost |
|---|---|---|---|---:|
| Tunnel A — Primary | 172.16.0.0/30 | 172.16.0.1 | 172.16.0.2 | 10 |
| Tunnel B — Backup | 172.16.0.4/30 | 172.16.0.5 | 172.16.0.6 | 100 |

Remote-access VPN address pool reserved in the design: `172.16.1.0/27`.

The two site-to-site tunnels and remote-access pool are design entries. Verify tunnel interfaces, IPsec status, routing, and failover before marking them completed.

## 10. Legacy RIP Domain

| Segment | Network | Purpose |
|---|---|---|
| Aggregate | 10.99.0.0/22 | Legacy routing domain |
| Link 1 | 10.99.0.0/30 | Point-to-point |
| Link 2 | 10.99.0.4/30 | Point-to-point |
| Legacy LAN 1 | 10.99.1.0/27 | Legacy LAN |
| Legacy LAN 2 | 10.99.2.0/27 | Legacy LAN |

The design places RIP-to-OSPF and OSPF-to-RIP redistribution on ALEX-ASBR only. Route tagging and filtering are intended to prevent routing loops. The peer addressing and actual redistribution configuration remain to be verified.

## 11. Management Network — Updated Design

The previous management network `192.168.1.16/28` is **removed from this project documentation**. The previously listed management addresses `.17`, `.18`, `.19`, and `.20` are no longer part of the current management plan.

The updated management network is:

| Item | Value |
|---|---|
| Network | 192.168.22.0/24 |
| Subnet Mask | 255.255.255.0 |
| Address 1 | 192.168.22.100 |
| Address 2 | 192.168.22.110 |
| Address 3 | 192.168.22.120 |

The device-to-address mapping, gateway, VLAN, and actual interface assignments have not yet been confirmed. Do not assume these addresses belong to particular FortiGate units or management PCs until the current VMware and lab settings are checked.

## 12. VMware Workstation Networking

The lab uses VMware Workstation with `VMnet8` recorded as part of the environment.

| Item | Value |
|---|---|
| Virtual network | VMnet8 |
| Typical VMware network type | NAT |
| Actual subnet | Verify in VMware Virtual Network Editor |
| VMnet8 host adapter address | Verify |
| DHCP enabled/disabled | Verify |
| Windows Server VM connectivity | Verify |

VMnet8 is the VMware virtual network used by the lab environment. Its subnet and host-side addresses must be copied from the actual VMware configuration; they must not be inferred from the project’s other IP ranges.

The previously documented `INET-HOST LAN` (`192.0.2.0/28`) is removed because that host was not added to the lab.

## 13. Windows Server and DHCP

The handoff records:

| Server | Address | Notes |
|---|---|---|
| DC01 | 192.168.0.36/28 | Gateway 192.168.0.33; `test.local` domain |
| DC02 | 192.168.0.37/28 | Gateway 192.168.0.33; DNS recorded as 192.168.0.36 |

The project includes Windows Server, Active Directory, DNS, and planned DHCP scopes and relay configuration.

DHCP scope ranges must respect each VLAN subnet and exclude gateways, server addresses, SVI addresses, and other static assignments. DHCP failover, directory replication, and DNS health must be verified separately.

## 14. Routing and Redundancy Design

The intended routing architecture includes:

- Cairo core VLAN routing in OSPF Area 1
- Cairo core-to-firewall transit links in Area 0
- Alexandria local routing in Area 2
- Alexandria firewall-facing link in Area 0
- Route-based IPsec Tunnel A and Tunnel B between sites
- OSPF summary for Cairo: `192.168.0.0/25`
- OSPF summary for Alexandria: `10.10.0.0/24`
- HSRP for redundant client default gateways
- FortiGate Active-Passive HA in Cairo
- Dual ISP connectivity through ISP-A and ISP-B

This section describes the intended design. Actual adjacency, route propagation, HA failover, and dual-WAN failover require verification evidence.

## 15. Removed or Superseded Entries

The following items must not be treated as part of the current design:

- The old management network `192.168.1.16/28` and its `.17`–`.20` assignments
- The `INET-HOST LAN` `192.0.2.0/28`, because the host was not added
- Any assumption that the management network addresses are automatically assigned to particular devices
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
- [ ] Confirm the actual management network implementation at `192.168.22.0/24`.
- [ ] Confirm the VMnet8 subnet and DHCP/NAT configuration.
- [ ] Verify Windows Server, AD DS, DNS, and DHCP functionality.

---

**Project status:** This document consolidates the supplied addressing plan, running configurations, and user-confirmed removals/additions. Values explicitly marked for verification must remain unverified until checked in the live lab.
