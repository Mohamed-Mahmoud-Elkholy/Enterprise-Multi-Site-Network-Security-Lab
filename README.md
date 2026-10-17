# Enterprise Multi-Site Network & Security Infrastructure Lab

A virtual enterprise networking and security project built with **GNS3, Cisco IOS/IOU, FortiGate, Windows Server, and VMware Workstation**.

The lab models an enterprise network connecting Cairo Headquarters (HQ) and Alexandria Branch, focusing on network segmentation, dynamic routing, firewall integration, dual-WAN connectivity, site-to-site IPsec VPNs, and infrastructure services.

## Network Topology

![Enterprise Multi-Site Network Topology](Images/Topology-overview.png)

*Enterprise network topology showing the Cairo HQ and Alexandria Branch infrastructure.*

## Project Objectives

- Design a multi-site enterprise network with structured IP addressing and VLAN segmentation.
- Implement Layer 2 and Layer 3 switching concepts.
- Configure OSPF routing between enterprise network segments.
- Integrate FortiGate firewalls into the network architecture.
- Design primary and backup Internet paths using dual-WAN connectivity.
- Configure site-to-site IPsec VPN tunnels between the two sites.
- Integrate Windows Server infrastructure services.
- Collect configuration and verification evidence.
- Investigate routing, connectivity, and redundancy issues using device CLI output and traffic diagnostics.

## Technologies

| Technology | Purpose |
|---|---|
| GNS3 | Network topology simulation |
| Cisco IOS / IOU | Switching, routing, and network protocols |
| VLANs and 802.1Q | Network segmentation |
| HSRP | Default-gateway redundancy |
| EtherChannel | Link aggregation |
| OSPF | Dynamic routing |
| RIP / OSPF Redistribution | Legacy routing integration; exact redistribution configuration requires further verification |
| FortiGate | Firewalling, routing, VIPs, and IPsec VPN |
| Dual-WAN | Primary and backup WAN paths |
| Windows Server | Directory and network infrastructure services |
| VMware Workstation / VMnet8 | Virtualization and lab connectivity |

## Network Architecture

### Cairo Headquarters

The HQ design includes:

- `HQ-CORE-01` and `HQ-CORE-02` as the core switching layer.
- `FG-HQ-01` and `FG-HQ-02` as the FortiGate firewall pair.
- Department-based VLANs and Layer 3 interfaces.
- HSRP and EtherChannel as part of the switching design.
- Two simulated ISP paths.
- Internal server connectivity.
- A DMZ web server at `192.168.0.146`, with HTTPS VIP configurations for WAN1 and WAN2.

External HTTPS access through both VIPs has not been fully validated.

### Alexandria Branch

The branch design includes:

- `FG-ALEX-01` as the branch firewall.
- `ALEX-CORE` as the branch core switch.
- Local VLANs and OSPF routing.
- Planned integration with a legacy RIP routing domain through `ALEX-ASBR`.

The exact RIP/OSPF redistribution configuration requires further verification.

### Inter-Site Connectivity

The design includes two route-based IPsec tunnels between Cairo and Alexandria. Collected FortiGate outputs showed active Security Associations and packet counters for both configured tunnels.

These observations confirm tunnel activity at the time of capture, but do not independently prove end-to-end application connectivity or successful automatic failover.

## Documentation

| Document | Description |
|---|---|
| [FortiGate HA Verification](Verification/fortigate-ha-verification.md) | HA roles, synchronization, heartbeat status, and collected failover evidence |
| [IP Addressing and VLANs](Documentation/ip-addressing-vlans.md) | VLAN IDs, subnets, and gateway addressing |
| [Network Topology and Architecture](Documentation/network-topology.md) | Site architecture, device roles, and routing design |
| [FortiGate Firewall Policies](Documentation/fortigate-firewall-policies.md) | Firewall policy inventory, DMZ VIP configuration, and observed connectivity results |
| [Routing Verification](Verification/routing.md) | Collected routing and OSPF evidence |
| [IPsec VPN Verification](Verification/ipsec-vpn.md) | Observed IPsec activity and outstanding VPN tests |
| [Dual-WAN Verification](Verification/dual-wan-verification.md) | WAN addressing, routing behavior, and failover evidence |

## Verification Evidence

Verification statements are based on command output and GUI evidence collected from the running lab. Results apply only to the specific tests performed at the time of capture.

### OSPF and Routing

Collected outputs from `HQ-CORE-01`, `FG-HQ-01`, and `FG-ALEX-01` showed:

- OSPF neighbors reaching `FULL` or `Full` state on the displayed adjacencies.
- A default route on `HQ-CORE-01` pointing to `192.168.0.130`.
- Routes to remote Alexandria networks.
- An OSPF External Type 2 route for `10.99.0.0/22`.
- OSPF neighbor relationships on the displayed IPsec tunnel interfaces.

The external route confirms that an OSPF external route existed in the routing table at the time of capture. It does not independently identify the source protocol or prove the exact redistribution configuration.

These observations do not, by themselves, prove end-to-end connectivity or complete routing redundancy.

### IPsec VPN

Collected FortiGate outputs showed active Security Associations and encapsulation/decapsulation packet counters for both configured tunnels.

This is evidence of observed tunnel activity, not proof that application traffic successfully traverses both paths or that automatic failover and recovery work as intended.

### DMZ and HTTPS Verification

The following tests were performed during troubleshooting:

| Test | Observed Result |
|---|---|
| ICMP from `FG-HQ-01` to `192.168.0.146` | Passed: 5 packets received, 0% packet loss |
| TCP connection from `FG-HQ-01` to `192.168.0.146:443` | Connection established, then closed by the remote host |
| External TCP test to WAN1 VIP `198.51.100.2:443` from `INET-EDGE` | TCP connection opened; WAN1 HTTPS policy traffic was observed in the logs |
| External TCP test to WAN2 VIP `203.0.113.2:443` from `INET-EDGE` | Timed out |
| WAN2 debug flow | DNAT to `192.168.0.146:443` was identified, followed by `reverse path check fail` and packet drop |

The WAN2 test confirms that the connection attempt reached `FG-HQ-01` through `port2` and that destination NAT was applied. The debug output then showed a reverse-path check failure.

The underlying cause has not yet been confirmed. Routing and reverse-path behavior require further investigation before changing the configuration.

Successful ICMP reachability and TCP connection establishment do not prove that the web application returns a valid HTTPS response. External HTTPS access through both VIPs remains incompletely verified.

## Evidence Images

### Dual-WAN

- [Failover to WAN2](Images/Dual-WAN-Failover-to-WAN2.png)
- [Link Monitor During Failover](Images/Dual-WAN-Monitor-During-Failover.png)
- [WAN1 Recovery Ping](Images/Dual-WAN-Ping-WAN1-Recovery.png)
- [WAN2 Connectivity Test](Images/Dual-WAN-Ping-WAN2-Success.png)
- [Recovery to WAN1](Images/Dual-WAN-Recovery-to-WAN1.png)

### FortiGate HA

- [HA Synchronization Status](Images/FG-HQ-HA-Status-In-Sync.png)
- [HA Primary Status](Images/FG-HQ-HA-Status-Primary.png)
- [HA Secondary Status](Images/FG-HQ-HA-Status-Secondary.png)
- [Observed HA Role Change to FG-HQ-2](Images/FG-HQ-HA-Failover-to-HQ-2.png)
- [HA Recovery Status](Images/FG-HQ-HA-Recovery.png)

### OSPF and Routing

- [HQ Core OSPF Interfaces](Images/HQ-CORE-1-OSPF-Interfaces.png)
- [HQ Core OSPF Neighbors](Images/HQ-CORE-1-OSPF-Neighbors.png)
- [HQ Core Routing Table](Images/HQ-CORE-1-Routing-Table.png)

These images show the displayed lab state at the time of capture. Image filenames alone do not establish that every related failover or connectivity scenario passed.

## Remaining Validation

The following items remain incomplete or require additional evidence:

- End-to-end application connectivity between Cairo HQ and Alexandria.
- Automatic failover and recovery between the two IPsec tunnels.
- Complete dual-WAN failover and recovery validation.
- Review of FortiGate HA synchronization and failover evidence.
- External HTTPS access through both DMZ VIPs.
- Windows Server service validation.
- Confirmation of the source protocol and exact configuration used for RIP/OSPF redistribution.
- Investigation of the WAN2 reverse-path check failure.

No configuration change should be considered a verified fix until the relevant connectivity and routing tests have been repeated successfully.

## Repository Structure

```text
Enterprise-Multi-Site-Network-Security-Lab/
├── README.md
├── Images/
│   ├── .gitkeep
│   ├── Dual-WAN-Failover-to-WAN2.png
│   ├── Dual-WAN-Monitor-During-Failover.png
│   ├── Dual-WAN-Ping-WAN1-Recovery.png
│   ├── Dual-WAN-Ping-WAN2-Success.png
│   ├── Dual-WAN-Recovery-to-WAN1.png
│   ├── FG-HQ-HA-Failover-to-HQ-2.png
│   ├── FG-HQ-HA-Recovery.png
│   ├── FG-HQ-HA-Status-In-Sync.png
│   ├── FG-HQ-HA-Status-Primary.png
│   ├── FG-HQ-HA-Status-Secondary.png
│   ├── HQ-CORE-1-OSPF-Interfaces.png
│   ├── HQ-CORE-1-OSPF-Neighbors.png
│   ├── HQ-CORE-1-Routing-Table.png
│   └── Topology-overview.png
├── Documentation/
│   ├── fortigate-firewall-policies.md
│   ├── ip-addressing-vlans.md
│   └── network-topology.md
├── configs/
│   ├── .gitkeep
│   └── README.md
└── Verification/
    ├── dual-wan-verification.md
    ├── fortigate-ha-verification.md
    ├── ipsec-vpn.md
    └── routing.md
```

The `configs/` directory is reserved for sanitized device configurations collected from the running lab. No actual device configuration files have been added yet.

## Security and Lab Disclaimer

This is a virtual lab developed for educational purposes, technical practice, and professional portfolio demonstration.

- Public-looking IP addresses may belong to reserved documentation ranges used in the simulation.
- Device configurations must be sanitized before publication.
- Passwords, pre-shared keys, private keys, tokens, and other sensitive information must never be committed to the public repository.
- Features must not be described as operational or successfully tested without supporting evidence.
- Verification results represent the state observed during the recorded tests and may change as the lab configuration evolves.

## Project Status

This repository documents the network design, selected configuration details, and verification evidence collected so far.

Several routing, VPN, HA, dual-WAN, and DMZ behaviors still require complete validation. Known issues are documented conservatively rather than being presented as successful tests. Additional evidence will be added as the remaining tests are completed.
