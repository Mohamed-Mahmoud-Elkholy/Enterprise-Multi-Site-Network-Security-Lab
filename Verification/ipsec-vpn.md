# IPsec VPN Verification

## 1. Overview

This document records the implementation and verification of site-to-site IPsec VPN connectivity between Cairo Headquarters and Alexandria Branch.

The design includes two route-based IPsec VPN tunnels:

| Tunnel | Purpose | Configured Preference |
|---|---|---|
| `Cairo-Alex-WAN1` | Primary inter-site path | Preferred |
| `Cairo-Alex-WAN2` | Backup inter-site path | Secondary |

The lab uses both tunnels to provide redundant inter-site connectivity. Verification includes tunnel status, routing-table evidence, failover to the backup WAN path, and recovery to the preferred WAN path.

## 2. VPN Tunnel Status Before Failover

The collected FortiGate evidence shows both configured IPsec tunnels established before the failover test, with incoming and outgoing data counters present.

**Result: PASS — Both VPN tunnels were operational in the captured pre-failover state.**

Evidence:
- [VPN Tunnels Before Failover](../Images/VPN-Tunnels-Before-Failover.png)
- [VPN Routing Before Failover](../Images/VPN-Routing-Before-Failover.png)

## 3. Primary Routing Through WAN1

The captured FortiGate routing table showed the remote networks routed through `Cairo-Alex-WAN1`, with tunnel next hop `198.51.100.10`.

The default route was:

```text
S* 0.0.0.0/0 [10/0] via 198.51.100.1, port1
```

The evidence shows WAN1 selected as the preferred default route before failover.

**Result: PASS — WAN1 was the preferred path in the recorded routing state.**

Evidence:
- [VPN Routing — WAN1 Primary](../Images/VPN-Routing-WAN1-Primary.png)

## 4. Failover to WAN2

During the recorded failover test, WAN1 became unavailable while WAN2 remained operational.

The captured VPN and routing evidence shows the WAN1-down/WAN2-up state and the transition to the backup path.

The default route through WAN2 was:

```text
S* 0.0.0.0/0 [20/0] via 203.0.113.1, port2
```

Remote routes were also observed through `Cairo-Alex-WAN2`, with tunnel next hop `203.0.113.10`.

**Result: PASS — The recorded tests demonstrated failover from WAN1 to WAN2.**

Evidence:
- [VPN WAN1 Down, WAN2 Up](../Images/VPN-WAN1-Down-WAN2-Up.png)
- [VPN Failover — WAN1 Down, WAN2 Up](../Images/VPN-Failover-WAN1-Down-WAN2-Up.png)
- [VPN Failover to WAN2](../Images/VPN-Failover-to-WAN2.png)

## 5. Recovery to WAN1

After WAN1 was restored, the recorded routing table showed the preferred default route returning to WAN1:

```text
S* 0.0.0.0/0 [10/0] via 198.51.100.1, port1
```

The recovery evidence documents the return to the preferred WAN1 path.

**Result: PASS — Recovery to WAN1 was demonstrated in the recorded test.**

Evidence:
- [VPN Recovery to WAN1](../Images/VPN-Recovery-to-WAN1.png)

## 6. Routing and OSPF Evidence

The collected FortiGate routing tables showed remote networks through both tunnel interfaces in their respective captured states.

Examples of the observed remote networks include:

| Destination | Route Type |
|---|---|
| `10.10.0.0/24` | OSPF Inter-Area |
| `10.10.1.0/30` | OSPF |
| `10.10.1.4/30` | OSPF Inter-Area |
| `10.99.0.0/22` | OSPF External Type 2 |

The collected outputs also showed OSPF neighbor relationships over the displayed tunnel interfaces on `FG-ALEX-01`.

**Result: PASS — The recorded routing evidence supports the intended primary/backup VPN path selection.**

## 7. Evidence Inventory

| Evidence File | What It Documents |
|---|---|
| [VPN Tunnels Before Failover](../Images/VPN-Tunnels-Before-Failover.png) | Tunnel status before failover |
| [VPN Routing Before Failover](../Images/VPN-Routing-Before-Failover.png) | Routing state before failover |
| [VPN Routing — WAN1 Primary](../Images/VPN-Routing-WAN1-Primary.png) | WAN1 preferred routing state |
| [VPN WAN1 Down, WAN2 Up](../Images/VPN-WAN1-Down-WAN2-Up.png) | WAN1 failure and WAN2 availability |
| [VPN Failover — WAN1 Down, WAN2 Up](../Images/VPN-Failover-WAN1-Down-WAN2-Up.png) | Failover state shown in the VPN dashboard |
| [VPN Failover to WAN2](../Images/VPN-Failover-to-WAN2.png) | Transition to the backup WAN path |
| [VPN Recovery to WAN1](../Images/VPN-Recovery-to-WAN1.png) | Recovery to the preferred WAN1 path |

## 8. Conclusion

**PASS — VPN tunnel operation, routing through the primary and backup tunnel interfaces, failover to WAN2, and recovery to WAN1 were demonstrated by the recorded lab evidence.**

The collected screenshots and routing outputs document the expected path-selection behavior. This VPN work is treated as completed project functionality.

The separate external HTTPS publishing issue affecting WAN2-to-DMZ access is documented in `fortigate-firewall-policies.md` and does not change this VPN verification result.
