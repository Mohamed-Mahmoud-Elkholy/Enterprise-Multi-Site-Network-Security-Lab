# Enterprise Multi-Site Network Topology

## 1. Project Overview

This project simulates an enterprise network connecting Cairo Headquarters (HQ) and the Alexandria branch. The lab combines enterprise switching, dynamic routing, firewall security, redundant Internet connectivity, site-to-site VPNs, and Windows Server infrastructure.

The environment is built using GNS3, Cisco IOS devices, FortiGate virtual appliances, VMware Workstation, and Windows Server virtual machines.

The design includes:

- Redundant core switching at Cairo HQ.
- VLAN segmentation and inter-VLAN routing.
- HSRP for gateway redundancy.
- EtherChannel and Spanning Tree Protocol.
- OSPF routing, including multiple areas and route redistribution.
- Dual ISP connectivity at Cairo HQ.
- FortiGate Active-Passive High Availability.
- Two WAN-associated site-to-site IPsec VPN paths.
- A DMZ web server.
- Windows Server roles for domain services, DNS, DHCP, and related administration.
- A separate Alexandria branch network.

The topology and configuration evidence document the intended design and the observed lab state. A component being shown in the topology does not independently prove that every associated function has passed testing.

## 2. Network Architecture

The lab is divided into the following logical components.

### 2.1 Cairo Headquarters

Cairo HQ contains the core switching layer, internal VLANs, FortiGate security appliances, two ISP connections, and the DMZ network.

The core switching layer provides internal routing and connects to the FortiGate appliances through the recorded transit network.

The FortiGate layer provides:

- Firewall policy enforcement.
- Internet access through WAN1 and WAN2.
- Routing between the internal network, WAN connections, and supported VPN paths.
- Site-to-site VPN connectivity.
- DMZ access control.
- Active-Passive HA.

### 2.2 Alexandria Branch

The Alexandria branch contains its own FortiGate appliance and internal network infrastructure. Its connectivity to Cairo uses the two recorded WAN-associated VPN paths.

The project includes routing and connectivity evidence for the branch-to-HQ design. Individual application flows should be considered verified only when supported by their specific test results.

### 2.3 Internet Edge

The simulated Internet edge contains two ISP paths.

- ISP-A provides the primary WAN1 path.
- ISP-B provides the backup WAN2 path.
- The `INET-EDGE` router provides the external test point used in the recorded Internet and DMZ publishing tests.

The lab uses documentation-range addresses for its simulated external networks. These addresses represent the lab topology, not publicly routable production Internet addresses.

## 3. Cairo HQ VLAN Design

The recorded HQ core configuration includes the following VLANs:

| VLAN ID | Name / Purpose | Recorded Network |
|---|---|---|
| 10 | USERS | `192.168.0.0/27` |
| 20 | SERVERS | `192.168.0.32/28` |
| 30 | IT | `192.168.0.48/28` |
| 40 | HR | `192.168.0.64/28` |
| 50 | FINANCE | `192.168.0.80/28` |
| 60 | SALES | `192.168.0.96/29` |
| 70 | GUEST | `192.168.0.104/29` |
| 99 | MANAGEMENT | `192.168.0.112/28` |
| 901 | FW-TRANSIT-A | `192.168.0.128/30` |

These entries reflect the recorded HQ core routing information. The project also contains a separate internal transit link with network `192.168.0.136/30` and a DMZ subnet `192.168.0.144/28`.

VLAN 901 is used for the recorded core-to-firewall transit connection. The complete physical and logical port assignments should be checked against the current GNS3 topology and switch configuration before making any changes.

The VLAN addressing plan is documented separately in [IP Addressing and VLANs](ip-addressing-vlans.md).

## 4. Core Switching and Gateway Redundancy

The HQ switching design uses two core switches and HSRP to provide redundant default gateways for the internal VLANs.

The recorded configuration includes:

- VLAN interfaces for the HQ networks.
- HSRP configuration for internal gateway redundancy.
- OSPF configuration on the core layer.
- EtherChannel and Spanning Tree Protocol as part of the switching design.

HSRP provides a virtual gateway for configured VLANs. Actual gateway availability during a core-switch failure must be established through a controlled test; configuration evidence alone does not prove uninterrupted client connectivity.

The project includes screenshots for the HQ core OSPF interfaces, configuration, neighbor state, routing table, and HSRP configuration.

## 5. OSPF and Routing Design

OSPF is used for routing between the documented network segments.

The recorded HQ core evidence includes an OSPF neighbor in the `FULL` state on `Vlan901`, with neighbor ID `3.3.3.3` and address `192.168.0.130`.

The routing evidence also includes remote routes for networks such as:

- `10.10.0.0/24`
- `10.10.1.0/30`
- `10.10.1.4/30`
- `10.99.0.0/22`
- `192.168.0.136/30`
- `192.168.0.144/28`

The route for `10.99.0.0/22` was displayed as an OSPF External Type 2 route.

The project design includes multiple OSPF areas and route redistribution. The specific source protocol and redistribution path for an external route must be confirmed from the relevant running configuration rather than inferred from the routing-table entry alone.

Detailed routing results are documented in [Routing Verification](../Verification/routing.md).

## 6. FortiGate Security Layer

### 6.1 Cairo FortiGate HA

Cairo HQ uses two FortiGate VM64-KVM appliances in Active-Passive mode:

- `FG-HQ-1`
- `FG-HQ-2`

The recorded normal cluster state showed `FG-HQ-1` as Primary and `FG-HQ-2` as Secondary, with both members synchronized.

The recorded interfaces include:

| Interface | Recorded Purpose | Recorded Address |
|---|---|---|
| `port1` | WAN1 / ISP-A | `198.51.100.2/29` |
| `port2` | WAN2 / ISP-B | `203.0.113.2/29` |
| `port5` | Inside-A | `192.168.0.130/30` |
| `port6` | Inside-B | `192.168.0.138/30` |
| `port7` | DMZ-HQ | DMZ interface |
| `port8` | Management | `192.168.22.100/24` |

The recorded HA heartbeat interfaces are `port3` and `port4`.

The recorded interface addresses and roles describe the lab state captured in the project evidence. Confirm the live interface configuration before changing any address or connection.

### 6.2 Alexandria FortiGate

The Alexandria branch uses `FG-ALEX-01` to enforce local firewall policies and connect the branch to its WAN paths and HQ VPN tunnels.

The project includes screenshots of Alexandria firewall policies, interface configuration, and IPsec tunnel status.

### 6.3 Firewall Policies

The firewall policy design includes:

- Internal-to-Internet policies for WAN1 and WAN2.
- Inter-site policies for the Cairo–Alexandria VPN paths.
- Internal access to the DMZ web server.
- External HTTPS publishing rules for the DMZ VIPs.

Policy configuration and the results of the corresponding connectivity tests are documented in [FortiGate Firewall Policies and Connectivity Verification](fortigate-firewall-policies.md).

## 7. Dual ISP and WAN Failover Design

Cairo HQ uses two simulated ISP connections.

| Parameter | ISP-A / WAN1 | ISP-B / WAN2 |
|---|---|---|
| Network | `198.51.100.0/29` | `203.0.113.0/29` |
| ISP gateway | `198.51.100.1` | `203.0.113.1` |
| FortiGate address | `198.51.100.2` | `203.0.113.2` |
| Default-route distance | 10 | 20 |

The captured routing tables showed WAN1 as the preferred default route and WAN2 as the backup route.

The recorded tests demonstrated:

1. Normal operation with WAN1 selected.
2. WAN1 monitor failure after the primary path was disconnected.
3. Default-route switching to WAN2.
4. Restoration of the WAN1 path.
5. Recovery of the preferred default route to WAN1.

These results establish the recorded default-route failover and recovery behavior. They do not establish uninterrupted application sessions during every transition.

See [Dual-WAN Failover and Recovery Verification](../Verification/dual-wan-verification.md).

## 8. Site-to-Site IPsec VPN Design

The lab contains two WAN-associated VPN paths between Cairo HQ and Alexandria:

- `Cairo-Alex-WAN1`
- `Cairo-Alex-WAN2`

The recorded routing evidence shows remote networks associated with the WAN1 tunnel path under the normal condition, the WAN2 tunnel path during failover, and the preferred WAN1 path after recovery.

The project includes tunnel-status screenshots from both FortiGate appliances and routing evidence for the recorded transitions.

These results demonstrate the recorded VPN routing failover and recovery. They do not independently prove zero packet loss or uninterrupted application sessions during a path change.

See [IPsec VPN Verification](../Verification/ipsec-vpn.md).

## 9. DMZ Web Server

The HQ DMZ contains the server named `WEB01-DMZ`.

| Parameter | Value |
|---|---|
| Server name | `WEB01-DMZ` |
| IP address | `192.168.0.146` |
| Subnet | `192.168.0.144/28` |
| Firewall interface | `DMZ-HQ (port7)` |
| Published service | HTTPS on TCP/443 |
| Internal services configured in policy | HTTP, HTTPS, and ICMP |

The recorded internal tests showed successful HTTP connectivity from `192.168.0.54` to `192.168.0.146:80` and successful ICMP reachability from the FortiGate to the server.

Internal TCP connectivity to port 443 produced mixed results in two recorded attempts.

External publishing through WAN1 produced a successful TCP connection, but the evidence did not establish a successful HTTPS application response.

External publishing through the WAN2 VIP remains unresolved. The captured debug flow showed destination NAT to `192.168.0.146:443` followed by `reverse path check fail`.

The cause of the WAN2 reverse-path failure has not been confirmed. It must not be described as fixed until the relevant configuration is reviewed and a new
