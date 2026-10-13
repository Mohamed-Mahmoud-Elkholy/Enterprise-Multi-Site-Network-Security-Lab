# Enterprise Multi-Site Network & Security Infrastructure Lab

A virtual enterprise networking and security project built with **GNS3, Cisco IOS/IOU, FortiGate, Windows Server, and VMware Workstation**.

The lab models a multi-site enterprise infrastructure connecting Cairo Headquarters (HQ) and Alexandria Branch, with a focus on network segmentation, dynamic routing, firewall integration, WAN connectivity, and infrastructure services.

## Network Topology

![Enterprise Multi-Site Network Topology](Images/topology-overview.png)

*Enterprise network topology showing the Cairo HQ and Alexandria Branch infrastructure.*

## Project Objectives

- Design a multi-site enterprise network with structured IP addressing and VLAN segmentation.
- Implement Layer 2 and Layer 3 switching concepts.
- Configure OSPF routing between enterprise network segments.
- Integrate FortiGate firewalls into the network architecture.
- Develop a dual-WAN design with primary and backup Internet paths.
- Design site-to-site IPsec VPN connectivity between the two sites.
- Integrate Windows Server infrastructure services.
- Collect configuration and verification evidence for technical review.

## Technologies

| Technology | Purpose |
|---|---|
| GNS3 | Network topology simulation |
| Cisco IOS / IOU | Switching, routing, and network protocols |
| VLANs and 802.1Q | Network segmentation |
| HSRP | Default-gateway redundancy design |
| EtherChannel | Link aggregation |
| OSPF | Dynamic routing |
| RIP / OSPF Redistribution | Planned integration with a legacy routing domain |
| FortiGate | Firewalling, routing, and IPsec VPN |
| Dual-WAN | Redundant WAN design |
| Windows Server | Directory and network infrastructure services |
| VMware Workstation / VMnet8 | Virtualization and lab Internet connectivity |

## Network Architecture

### Cairo Headquarters

The HQ design includes:

- `HQ-CORE-01` and `HQ-CORE-02` as the core switching layer.
- `FG-HQ-01` and `FG-HQ-02` as the FortiGate firewall pair.
- Department-based VLANs and Layer 3 interfaces.
- HSRP and EtherChannel as part of the switching design.
- Two simulated ISP paths.
- Internal server connectivity and a planned DMZ web server.

### Alexandria Branch

The branch design includes:

- `FG-ALEX-01` as the branch firewall.
- `ALEX-CORE` as the branch core switch.
- Local VLANs and OSPF routing.
- Planned integration with a legacy RIP routing domain through `ALEX-ASBR`.

### Inter-Site Connectivity

The design includes two route-based IPsec tunnels between Cairo and Alexandria. Both tunnels have displayed active Security Associations and packet counters in the collected FortiGate outputs.

This confirms observed IPsec SA activity, but does not by itself establish end-to-end application connectivity or successful failover.

## Documentation

| Document | Description |
|---|---|
| [IP Addressing and VLANs](Documentation/ip-addressing-vlans.md) | VLAN IDs, subnets, and gateway addressing |
| [Network Topology and Architecture](Documentation/network-topology.md) | Site architecture, device roles, routing design, and planned connectivity |
| [Routing Verification](Verification/routing.md) | Collected routing and OSPF evidence |

## Verification Evidence

Verification is based on command output collected from the running lab.

### OSPF and Routing

The collected outputs from `HQ-CORE-01`, `FG-HQ-01`, and `FG-ALEX-01` show:

- OSPF neighbors reaching `FULL` / `Full` state on the displayed adjacencies.
- A default route on `HQ-CORE-01` pointing to `192.168.0.130`.
- Routes to remote Alexandria networks and the legacy routing prefix.
- OSPF-learned routes for the inter-site tunnel endpoint addresses.
- OSPF neighbor relationships across the two displayed IPsec tunnel interfaces on `FG-ALEX-01`.

### IPsec Tunnels

The FortiGate tunnel outputs show active Security Associations and increasing encapsulation/decapsulation packet counters for both configured tunnels.

**Important:** These results are evidence of observed tunnel activity, not proof that the complete primary/backup failover process works correctly.

### Remaining Validation

The following require additional evidence before being described as successfully tested:

- End-to-end connectivity between sites.
- Automatic failover between the two IPsec tunnels.
- Dual-WAN Internet failover.
- FortiGate HA failover and synchronization.
- DMZ web server reachability.
- Windows Server service validation.
- RIP/OSPF redistribution behavior.

## Repository Structure

```text
Enterprise-Multi-Site-Network-Security-Lab/
├── README.md
├── Images/
│   └── Topology-overview.png
├── Documentation/
│   ├── ip-addressing-vlans.md
│   └── network-topology.md
├── configs/
└── Verification/
    └── routing.md
```

The structure above reflects the documented project layout. The contents of `configs/` should be added to this tree only after the actual configuration files have been collected and sanitized.

## Security and Lab Disclaimer

This is a virtual lab developed for educational purposes, technical practice, and professional portfolio demonstration.

- Public-looking IP addresses may belong to reserved documentation ranges used in the simulation.
- Device configurations must be sanitized before publication.
- Passwords, pre-shared keys, private keys, tokens, and other sensitive information must never be committed to the public repository.
- Features must not be described as operational or successfully tested without supporting evidence.

## Project Status

This repository documents the network design and the verification evidence collected so far. Implementation status is described conservatively, and additional test evidence will be added as the remaining functionality is validated.

