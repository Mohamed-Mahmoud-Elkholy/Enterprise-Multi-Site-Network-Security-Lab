# Enterprise Multi-Site Network & Security Infrastructure Lab
## IP Addressing, VLANs, Routing & Infrastructure

**Environment:** GNS3, Cisco IOS, FortiGate, Windows Server, VMware Workstation  
**Sites:** Cairo HQ and Alexandria Branch

> **Documentation policy:** This document separates design values, previously recorded observations, and items that still require verification. A planned IP address or historical command output does not prove that a service or path is currently operational.

## 1. Cairo HQ — VLAN and VLSM Plan

| VLAN ID | Name | Network | Subnet Mask | HSRP Virtual IP |
|---:|---|---|---|---|
| 10 | USERS | `192.168.0.0/27` | `255.255.255.224` | `192.168.0.1` |
| 20 | SERVERS | `192.168.0.32/28` | `255.255.255.240` | `192.168.0.33` |
| 30 | IT | `192.168.0.48/28` | `255.255.255.240` | `192.168.0.49` |
| 40 | HR | `192.168.0.64/28` | `255.255.255.240` | `192.168.0.65` |
| 50 | FINANCE | `192.168.0.80/28` | `255.255.255.240` | `192.168.0.81` |
| 60 | SALES | `192.168.0.96/29` | `255.255.255.248` | `192.168.0.97` |
| 70 | GUEST | `192.168.0.104/29` | `255.255.255.248` | `192.168.0.105` |
| 99 | MANAGEMENT | `192.168.0.112/28` | `255.255.255.240` | `192.168.0.113` |

The Cairo VLAN networks are summarized as `192.168.0.0/25` in the routing design. The HSRP virtual IP is the intended client default gateway. DHCP scopes must exclude virtual IPs, actual SVI addresses, servers, and other statically assigned infrastructure addresses.

## 2. Cairo HQ — Core Switching

Documented switching features:

- VLANs 10, 20, 30, 40, 50, 60, 70, and 99
- 802.1Q trunking
- EtherChannel using LACP
- Rapid-PVST spanning tree
- HSRP gateway redundancy
- Layer 3 SVIs and DHCP relay

### HSRP Status Recorded for HQ-CORE-01

| VLAN | HQ-CORE-01 Priority | Recorded State | Virtual IP |
|---:|---:|---|---|
| 10 | 110 | Active | `192.168.0.1` |
| 20 | 110 | Active | `192.168.0.33` |
| 30 | 110 | Active | `192.168.0.49` |
| 40 | 90 | Standby | `192.168.0.65` |
| 50 | 110 | Active | `192.168.0.81` |
| 60 | 90 | Standby | `192.168.0.97` |
| 70 | 110 | Active | `192.168.0.105` |
| 99 | 90 | Standby | `192.168.0.113` |

These are recorded observations, not a guarantee of the current state. Verify both core switches before documenting current HSRP roles.

## 3. Cairo HQ — Core and Firewall Transit

The current design retains two Core-to-FortiGate transit networks. The previously listed networks `192.168.0.132/30` and `192.168.0.140/30` have been removed.

| Link | Network | Core Address | FortiGate Address |
|---|---|---|---|
| `HQ-CORE-01` ↔ `FG-HQ` | `192.168.0.128/30` | CORE-1: `192.168.0.129` | `192.168.0.130` |
| `HQ-CORE-02` ↔ `FG-HQ` | `192.168.0.136/30` | CORE-2: `192.168.0.137` | `192.168.0.138` |

### Firewall Transit VLANs

| VLAN ID | Name | Core Switch |
|---:|---|---|
| 901 | FW-TRANSIT-A | `HQ-CORE-01` |
| 902 | FW-TRANSIT-B | `HQ-CORE-02` |

The supplied `HQ-CORE-01` configuration contains SVI VLAN 901 at `192.168.0.129/30` and a default route via `192.168.0.130`. Verify the actual `HQ-CORE-02` and FortiGate interface configurations and physical connections against the current lab. Do not restore the removed transit networks from older plans.

## 4. Cairo HQ — FortiGate HA

### HA Design

- Devices: `FG-HQ-01` and `FG-HQ-02`
- HA mode: Active-Passive
- Heartbeat interfaces: ports 3 and 4
- Previously recorded monitored interfaces: ports 1, 2, 5, and 6
- WAN interfaces: ports 1 and 2
- Core-facing interfaces: ports 5 and 6
- DMZ interface: port 7

The project handoff records Cairo FortiGate HA as configured. Historical status evidence showed both members in-sync, with HA health reported as OK during the recorded checks. Current HA health and controlled traffic failover still require verification.

### Firewall Loopbacks

| Device | Loopback Address |
|---|---|
| `FG-HQ-01` | `192.168.1.9/32` |
| `FG-HQ-02` | `192.168.1.10/32` |

The addressing plan places these addresses within `192.168.1.8/29`. Verify cluster router-ID and loopback behavior before treating the units as independent routing identities.

## 5. Cairo HQ — WAN, ISP and Internet

### FortiGate WAN Interfaces

| Interface | Network | FortiGate Address | ISP Gateway | Recorded VIP External Address |
|---|---|---|---|---|
| WAN1 / port1 | `198.51.100.0/29` | `198.51.100.2/29` | `198.51.100.1` | `198.51.100.2` |
| WAN2 / port2 | `203.0.113.0/29` | `203.0.113.2/29` | `203.0.113.1` | `203.0.113.2` |

The recorded firewall VIP configuration uses the WAN interface addresses as external VIP addresses:

- WAN1 VIP maps TCP port 443 on `198.51.100.2` to `192.168.0.146:443`.
- WAN2 VIP maps TCP port 443 on `203.0.113.2` to `192.168.0.146:443`.

These are recorded configuration details, not proof of successful external HTTPS access. The WAN2 troubleshooting evidence showed destination NAT followed by a reverse-path check failure and packet drop. External HTTPS access through both VIPs remains incompletely verified.

### ISP-A to INET

| Device / Interface | Address |
|---|---|
| Network | `198.51.100.248/30` |
| ISP-A Fa0/0 | `198.51.100.249` |
| INET Fa0/0 | `198.51.100.250` |

### ISP-B to INET

| Device / Interface | Address |
|---|---|
| Network | `203.0.113.248/30` |
| ISP-B Fa0/0 | `203.0.113.249` |
| INET Fa1/0 | `203.0.113.250` |

### WAN Monitoring

The recorded link-monitor configuration uses:

- WAN1 target: `198.51.100.250`
- WAN1 gateway: `198.51.100.1`
- WAN2 target: `203.0.113.250`
- WAN2 gateway: `203.0.113.1`

Historical evidence showed WAN1 and WAN2 monitoring configured, but the recorded WAN1 monitoring results included packet loss. The routing table also showed WAN1 preferred by route distance under the documented configuration.

Verify current link-monitor status, routing-table selection, and connectivity during WAN failure and recovery. A link monitor and an SD-WAN Performance SLA are different features.

## 6. DMZ and Public Web Service

| Item | Network / Address | Status |
|---|---|---|
| DMZ subnet | `192.168.0.144/28` | Addressing plan |
| FortiGate DMZ address | `192.168.0.145` | Recorded in plan |
| WEB01 | `192.168.0.146` | Recorded as the mapped web-server address; service status requires verification |

The DMZ is intended to host a web server reachable through controlled firewall policies and public VIP/NAT configuration.

### Address Conflict Warning

An earlier validation note also assigned `192.168.0.145` to `HQ-CORE-01` and `192.168.0.146` to an `INSIDE-A` interface. These addresses conflict with the recorded DMZ plan.

Check the actual topology and current interface configurations before making changes. Do not assign or move addresses until the connected interfaces and subnet usage have been reconciled.

## 7. Alexandria Branch — VLAN and IP Addressing

| VLAN ID | Name | Network | Subnet Mask | ALEX-CORE SVI |
|---:|---|---|---|---|
| 10 | USERS | `10.10.0.0/25` | `255.255.255.128` | `10.10.0.2` |
| 20 | SERVERS | `10.10.0.128/27` | `255.255.255.224` | `10.10.0.130` |
| 30 | SALES | `10.10.0.192/28` | `255.255.255.240` | `10.10.0.193` |
| 99 | MANAGEMENT | `10.10.0.160/28` | `255.255.255.240` | `10.10.0.162` |

### Alexandria Core Loopbacks and Routed Links

| Interface / Link | Address | Network | Status |
|---|---|---|---|
| `ALEX-CORE` Loopback0 | `10.10.0.177/32` | `10.10.0.176/29` allocation | Present in running configuration |
| `FG-ALEX` ↔ `ALEX-CORE` | Planned FG `.1`, Core `.2` | `10.10.1.0/30` | Address conflict; verify |
| `ALEX-CORE` ↔ `ALEX-ASBR` | Planned Core `.5`, ASBR `.6` | `10.10.1.4/30` | Core address present; peer unverified |
| `ALEX-ASBR` Loopback | `10.10.0.178/32` | `10.10.0.176/29` allocation | Design value; verify |

**Live-config discrepancy:** The supplied `ALEX-CORE` running configuration shows Ethernet0/0 at `10.10.1.1/30` and Ethernet0/1 at `10.10.1.5/30`. The handoff instead proposes `ALEX-CORE` `.2` on the first link and `.5` on the second. Confirm actual cable endpoints and peer configurations before changing anything.

The supplied `ALEX-CORE` output shows VLAN 10, 20, 30, and 99 SVIs as `down/down`. The VLANs exist, but operational Layer 3 connectivity was not demonstrated by that output.

## 8. Alexandria Branch — OSPF

The supplied `ALEX-CORE` configuration contains:

- OSPF process: 1
- Router ID: `5.5.5.5`
- Ethernet0/0 network `10.10.1.0/30` assigned to Area 0
- Ethernet0/1 network `10.10.1.4/30` assigned to Area 2
- Area 2 summary configured as `10.10.0.0/24`
- Default route configured through `10.10.1.2`

The intended design treats `ALEX-CORE` as an Area Border Router and `ALEX-ASBR` as the redistribution point for the legacy RIP domain. Verify OSPF neighbors, learned routes, summary advertisement, and default-route reachability before claiming the design is operational.

## 9. Cairo–Alexandria Site-to-Site VPN

| Tunnel | Network | Cairo Address | Alexandria Address | Planned Cost |
|---|---|---|---|---:|
| Tunnel A — Primary | `172.16.0.0/30` | `172.16.0.1` | `172.16.0.2` | 10 |
| Tunnel B — Backup | `172.16.0.4/30` | `172.16.0.5` | `172.16.0.6` | 100 |

Remote-access VPN address pool reserved in the design: `172.16.1.0/27`.

Historical notes recorded active IPsec SAs, encapsulation/decapsulation counters, and OSPF neighbors over tunnel interfaces on `FG-ALEX-01`. Those observations indicate past tunnel activity, but do not by themselves prove current end-to-end reachability or failover.

## 10. Legacy RIP Domain

| Segment | Network | Purpose |
|---|---|---|
| Aggregate | `10.99.0.0/22` | Legacy routing domain |
| Link 1 | `10.99.0.0/30` | Point-to-point |
| Link 2 | `10.99.0.4/30` | Point-to-point |
| Legacy LAN 1 | `10.99.1.0/27` | Legacy LAN |
| Legacy LAN 2 | `10.99.2.0/27` | Legacy LAN |

The design places RIP-to-OSPF and OSPF-to-RIP redistribution on `ALEX-ASBR` only. Route tagging and filtering are intended to prevent routing loops. Peer addressing and actual redistribution configuration remain to be verified.

## 11. Management Network — Updated

The previous management network `192.168.1.16/28` and its `.17`–`.20` assignments have been removed from the current plan.

| Item | Value |
|---|---|
| Network | `192.168.22.0/24` |
| Subnet Mask | `255.255.255.0` |
| Reserved address 1 | `192.168.22.100` |
| Reserved address 2 | `192.168.22.110` |
| Reserved address 3 | `192.168.22.120` |

The device-to-address mapping, gateway, VLAN, and interface assignments have not been specified. Do not assign these addresses to particular devices without confirmation.

## 12. Internet Access and VMware VMnet8

**VMnet8 represents the lab's Internet connection.** It is the external Internet/NAT network used by the lab environment. No additional INET-HOST device or separate `192.0.2.0/28` host LAN was added.

- VMware network: `VMnet8`
- Role in this lab: Internet connectivity for devices that are routed and permitted to access the Internet
- No separate INET-HOST node or LAN is part of the current design
- No additional VMnet8 subnet or host IP addresses are specified in this document

## 13. Windows Server — Three-Server Inventory and Comparison

This section uses the same fields for all three Windows servers. Unknown Alexandria values are not copied from Cairo or guessed.

### 13.1 Side-by-Side Comparison

| Field | `DC01` — Cairo HQ | `DC02` — Cairo HQ | `ALEX-DC-01` — Alexandria Branch |
|---|---|---|---|
| Hostname | `DC01` | `DC02` | `ALEX-DC-01` in topology; `ALEX-DC1` appears as shorthand in the plan |
| Site | Cairo HQ | Cairo HQ | Alexandria Branch |
| Intended VLAN | VLAN 20 — SERVERS | VLAN 20 — SERVERS | VLAN 20 — SERVERS |
| Server subnet | `192.168.0.32/28` | `192.168.0.32/28` | `10.10.0.128/27` |
| Recorded host IP | `192.168.0.36/28` | `192.168.0.37/28` | Not recorded |
| Gateway | `192.168.0.33` | `192.168.0.33` | Planned VLAN gateway `10.10.0.130`; host configuration unverified |
| DNS client setting | Not independently recorded; verify live | `192.168.0.36` recorded | Not recorded |
| AD DS / domain | `test.local` recorded; confirm live DC role | Cairo DC pair in project history; verify current role and state | Planned child-domain server; child-domain FQDN and promotion state not recorded |
| DNS server role | Windows DNS is part of project; current role and zone health unverified | Windows DNS is part of project; current role and zone health unverified | Not confirmed |
| DHCP role | Cairo Windows DHCP environment recorded; exact ownership/scopes need verification | DHCP/failover is in project history; exact partner state needs verification | Not confirmed |
| DHCP failover | Not proven; verify role and relationship | Not proven; verify role and relationship | Not recorded |
| Evidence status | IP, gateway, and domain values documented; current health unproven | IP, gateway, and DNS values documented; current health unproven | Hostname and intended subnet documented; host IP and service roles unverified |
| Next verification | Check IP, AD DS, DNS, DHCP, replication, and FSMO roles | Check IP, AD DS, DNS, DHCP/failover, replication, and FSMO roles | Verify OS/IP first; confirm VPN path to Cairo before child-domain promotion |

### 13.2 Cairo DC01 and DC02 — Recorded Configuration

| Server | IP / Prefix | Default Gateway | DNS setting recorded | Domain information |
|---|---|---|---|---|
| `DC01` | `192.168.0.36/28` | `192.168.0.33` | Not independently recorded | `test.local` |
| `DC02` | `192.168.0.37/28` | `192.168.0.33` | `192.168.0.36` | Cairo domain-controller pair in project history; verify live |

Both addresses belong to Cairo VLAN 20, `192.168.0.32/28`, with intended HSRP gateway `192.168.0.33`. These values do not prove AD replication, DNS health, DHCP authorization, or DHCP failover.

### 13.3 Alexandria ALEX-DC-01 — Build and Promotion Boundary

The project plan says to prepare Alexandria's Windows OS and IP configuration first. Child-domain promotion should happen only after the site-to-site tunnels are established and a working path to Cairo DC01 exists.

Known from project records:

- Topology hostname: `ALEX-DC-01`.
- Intended server VLAN: Alexandria VLAN 20, network `10.10.0.128/27`.
- Planned gateway for that VLAN: `10.10.0.130` on `ALEX-CORE`.
- Exact server IP, host subnet mask, DNS settings, domain suffix, child-domain FQDN, and current AD DS/DNS/DHCP role state are **not recorded**.
- `ALEX-DC1` appears in the plan as a naming variation; do not assume it is a separate server.

Do not promote the server based on the design alone. First verify its actual IP configuration and the VPN/routing path. Confirm the parent/child domain naming and DNS design before promotion. Do not assign an IP until it has been checked against the live lab and DHCP exclusions.

### 13.4 Windows Server Verification Commands

Run read-only checks first and review actual outputs.

**On each server:**

```powershell
ipconfig /all
hostname
Get-WindowsFeature AD-Domain-Services,DNS,DHCP
```

**On the Cairo domain-controller environment, where the tools are installed:**

```powershell
dcdiag /v
repadmin /replsummary
repadmin /showrepl
netdom query fsmo
nslookup -type=SRV _ldap._tcp.dc._msdcs.test.local
Get-DnsServerZone
```

**On a DHCP server, if the DHCP role is installed:**

```powershell
Get-DhcpServerInDC
Get-DhcpServerv4Scope
Get-DhcpServerv4Failover
```

Use commands only where the relevant roles/tools are installed. These commands are verification steps, not proof of success until their outputs are collected. Do not use an invented child-domain name for Alexandria checks.

## 14. Routing and Redundancy — Design vs Recorded Evidence

### 14.1 Intended Multi-Area OSPF Design

| Site / Device | Intended OSPF area(s) | Intended role | Evidence status |
|---|---|---|---|
| Cairo `HQ-CORE-01` and `HQ-CORE-02` | Areas 0 and 1 | ABRs; transit in Area 0, Cairo VLAN SVIs in Area 1 | Partial historical evidence; current configs and both adjacencies need verification |
| Cairo `FG-HQ` cluster | Area 0 | Single logical firewall routing identity | Interface and cluster router-ID details need verification |
| Alexandria `FG-ALEX-01` | Area 0 | Backbone router on firewall-facing link and site-to-site tunnels, as designed | Historical tunnel OSPF activity recorded; current state unverified |
| Alexandria `ALEX-CORE` | Areas 0 and 2 | ABR; VLANs in Area 2 and firewall-facing link in Area 0 | Configuration recorded; SVI and neighbor state need verification |
| Alexandria `ALEX-ASBR` | Area 2 for OSPF; RIP on legacy side | Sole planned RIP/OSPF redistribution point | Redistribution and peer status unverified |
| `LEG-R1`, `LEG-R2` | RIPv2 only | Legacy routing domain | Current addresses and state unverified |

The intended summaries are `192.168.0.0/25` for Cairo's VLAN block in Area 1 and `10.10.0.0/24` for Alexandria's local networks in Area 2. The Alexandria `/24` summary must not accidentally cover routed link `10.10.1.0/30`.

### 14.2 Cairo OSPF — Historical Evidence

Previously recorded `HQ-CORE-01` output showed:

| Observation | Previously recorded result | What it proves |
|---|---|---|
| Neighbor on `Vlan901` | Neighbor ID `3.3.3.3`, address `192.168.0.130`, state `FULL/DR` | Historical adjacency on Core-to-FortiGate transit; not proof of current state |
| Other neighbor | Neighbor ID `2.2.2.2` seen on local VLAN interfaces with `FULL/DR` | Needs reconciliation with intended passive-SVI design |
| Default route | Via `192.168.0.130` | Historical route selection; verify current routing table |
| Learned routes | `10.10.0.0/24`, `10.10.1.0/30`, `10.10.1.4/30`, `10.99.0.0/22`, `192.168.0.136/30`, `192.168.0.144/28` | Routes were present in the captured output; end-to-end reachability is not proven |
| Cairo summary | `192.168.0.0/25` via `Null0` | Historical summary route; verify current config and advertisement |

**Important discrepancy:** The design says user/server VLAN SVIs should be passive in OSPF, but historical output recorded neighbors on local VLAN interfaces. Do not silently label this correct or change the configuration without reviewing current `HQ-CORE-01` and `HQ-CORE-02` configurations and OSPF outputs.

### 14.3 Redundancy Goals and Required Proof

| Function | Intended design | Evidence required before calling it verified |
|---|---|---|
| Core gateway redundancy | HSRP across `HQ-CORE-01` and `HQ-CORE-02` | Current HSRP state and controlled gateway failover test |
| Cairo firewall redundancy | `FG-HQ-01` / `FG-HQ-02` Active-Passive HA | Current cluster health and controlled HA failover test |
| Dual Internet paths | WAN1 and WAN2 through ISP-A and ISP-B | Current routes/monitor status, WAN1 failure and recovery test |
| Site-to-site redundancy | Tunnel A primary, Tunnel B backup; planned costs 10 and 100 | End-to-end tests before, during, and after controlled primary-tunnel outage |
| Inter-site routing | OSPF over site-to-site design | Neighbor state, learned routes, preference, and end-to-end tests |
| RIP/OSPF redistribution | `ALEX-ASBR` only, with tagging/filtering | Current config, tags, learned routes, and loop-prevention validation |

### 14.4 Routing Evidence Rules

- Keep **Configured**, **Tested**, and **Verified** as separate states.
- An OSPF neighbor in `FULL` proves adjacency at the time of the output, not application reachability.
- A route in the routing table proves route installation, not successful forwarding.
- Active IPsec SAs and increasing counters prove tunnel activity at the time observed, not successful failover.
- Do not change IP addresses, OSPF areas, summaries, or cable mapping until current configurations and connected peers have been reconciled.

## 15. Removed or Superseded Entries

The following items are not part of the current design:

- Old management network `192.168.1.16/28` and addresses `.17`–`.20`
- Transit network `192.168.0.132/30`
- Transit network `192.168.0.140/30`
- Separate `INET-HOST LAN` `192.0.2.0/28`
- A separate INET-HOST node
- Removed HQ access switches `HQ-SW-SALES` and `HQ-SW-AC`
- Assumptions that new management addresses belong to specific devices without confirmation
- Claims that WEB01, public VIPs, remote-access VPN, routing failover, or Windows Server services are operational without current verification

## 16. Verification Checklist

### Switching and Firewall

- [ ] Confirm `HQ-CORE-01` and `HQ-CORE-02` transit interfaces and addresses.
- [ ] Verify current HSRP roles on both Cairo core switches.
- [ ] Resolve the DMZ `.145` and `.146` addressing conflict.
- [ ] Confirm FortiGate HA status and monitored interfaces.
- [ ] Verify WAN1 and WAN2 routing and failover.
- [ ] Verify Alexandria routed links against actual connected devices.
- [ ] Verify Alexandria SVIs become operational.

### Routing and VPN

- [ ] Verify OSPF neighbors, learned routes, summaries, and passive-interface behavior.
- [ ] Reconcile historical Cairo OSPF neighbors on local VLANs with the intended design.
- [ ] Verify both IPsec tunnels and route preference.
- [ ] Verify end-to-end Cairo–Alexandria reachability and tunnel failover/recovery.
- [ ] Verify RIP redistribution, route tags/filtering, and loop prevention.
- [ ] Confirm implementation of management network `192.168.22.0/24`.
- [ ] Verify Internet access through VMware VMnet8.

### Windows Servers

- [ ] Verify `DC01` IP, gateway, DNS client settings, AD DS, DNS, and DHCP roles.
- [ ] Verify `DC02` IP, gateway, DNS client settings, AD DS, DNS, DHCP/failover roles, and partner relationship.
- [ ] Verify AD replication, DNS SRV records, and FSMO role ownership for the Cairo domain.
- [ ] Verify DHCP scopes, exclusions, relay, and client gateway/DNS options.
- [ ] Verify `ALEX-DC-01` hostname and actual OS/IP configuration.
- [ ] Confirm a working routed/VPN path from Alexandria to Cairo DC01 before child-domain promotion.
- [ ] Confirm the intended child-domain FQDN and DNS design before promotion.
- [ ] Do not mark Alexandria AD DS/DNS/DHCP roles as installed until confirmed.

### GitHub Documentation

- [ ] Confirm every Markdown file is in the correct directory.
- [ ] Check README links and topology image rendering.
- [ ] Add configuration files only after collecting and sanitizing actual device configurations.
- [ ] Add screenshots and verification evidence together after Markdown documentation is complete.

---

**Project status:** This document consolidates the supplied addressing plan, historical configuration evidence, and user-confirmed removals/additions. Unknown values are explicitly marked instead of guessed. Verify each item against the live lab before marking it complete.
