# Routing Verification

## 1. Test Environment

- **Core device:** `HQ-CORE-01`
- **Firewall:** `FG-HQ-1`
- **Routing protocol:** OSPF
- **Verification commands:**
  - `show ip ospf neighbor`
  - `show ip route`
  - `get router info routing-table all`

This document records routing evidence collected from the running lab. Results describe the device state at the time of capture. Routing-table evidence is distinguished from end-to-end application testing.

## 2. OSPF Neighbor Verification

The `show ip ospf neighbor` output from `HQ-CORE-01` showed:

- Neighbor ID `3.3.3.3` on `Vlan901`, address `192.168.0.130`, state `FULL/DR`.
- Neighbor ID `2.2.2.2` appeared on the local VLAN interfaces, with the displayed neighbor state `FULL/DR`.

**Result: PASS.** The displayed OSPF adjacencies were in the `FULL` state during the recorded check.

These results confirm the reported neighbor states at that time. Application connectivity remains a separate test.

## 3. HQ Core Routing Table

### 3.1 Default Route

The captured routing table showed:

```text
S* 0.0.0.0/0 via 192.168.0.130
```

The default route points toward `192.168.0.130` through the FortiGate transit connection.

**Result: PASS.** A static default route toward the FortiGate was present in the routing table.

Internet failover and recovery are documented separately in [Dual-WAN Verification](dual-wan-verification.md).

### 3.2 Remote Routes

| Destination | Route Type | Next Hop | Interface |
|---|---|---|---|
| `10.10.0.0/24` | OSPF Inter-Area | `192.168.0.130` | `Vlan901` |
| `10.10.1.0/30` | OSPF | `192.168.0.130` | `Vlan901` |
| `10.10.1.4/30` | OSPF Inter-Area | `192.168.0.130` | `Vlan901` |
| `10.99.0.0/22` | OSPF External Type 2 | `192.168.0.130` | `Vlan901` |
| `192.168.0.136/30` | OSPF | `192.168.0.130` | `Vlan901` |
| `192.168.0.144/28` | OSPF | `192.168.0.130` | `Vlan901` |

These entries document the remote routes displayed by `HQ-CORE-01` at the time of capture.

**Result: PASS.** The listed remote routes were present in the routing table.

### 3.3 Local Networks

The routing table showed the following Cairo VLAN networks as directly connected:

| VLAN | Network |
|---|---|
| VLAN 10 | `192.168.0.0/27` |
| VLAN 20 | `192.168.0.32/28` |
| VLAN 30 | `192.168.0.48/28` |
| VLAN 40 | `192.168.0.64/28` |
| VLAN 50 | `192.168.0.80/28` |
| VLAN 60 | `192.168.0.96/29` |
| VLAN 70 | `192.168.0.104/29` |
| VLAN 99 | `192.168.0.112/28` |

The summary route `192.168.0.0/25` was also present with a `Null0` interface.

The connected routes document the local networks represented in the captured routing table. The purpose of the `Null0` summary route should be interpreted alongside the relevant routing configuration.

## 4. OSPF Route Redistribution Evidence

The routing table on `HQ-CORE-01` displayed:

| Field | Value |
|---|---|
| Destination | `10.99.0.0/22` |
| Route Type | OSPF External Type 2 |
| Next Hop | `192.168.0.130` |
| Interface | `Vlan901` |

**Result: PASS.** An OSPF External Type 2 route for `10.99.0.0/22` was present in the routing table.

The route entry alone does not identify the original source protocol or prove the complete redistribution configuration. Those details must be established from the relevant routing configuration.

## 5. FortiGate VPN Routing Evidence

The captured `FG-HQ-1` routing tables provide evidence of route selection through the two Cairo–Alexandria VPN tunnels.

### 5.1 WAN1-Associated Tunnel Selected

One captured routing table showed the following routes through `Cairo-Alex-WAN1`:

| Destination | Route Type | Tunnel Next Hop |
|---|---|---|
| `10.10.0.0/24` | OSPF Inter-Area | `198.51.100.10` |
| `10.10.1.0/30` | OSPF | `198.51.100.10` |
| `10.10.1.4/30` | OSPF Inter-Area | `198.51.100.10` |
| `10.99.0.0/22` | OSPF External Type 2 | `198.51.100.10` |

The same routing output showed the default route through WAN1:

```text
S* 0.0.0.0/0 [10/0] via 198.51.100.1, port1
```

### 5.2 WAN2-Associated Tunnel Selected

Another captured routing table showed the remote routes through `Cairo-Alex-WAN2`:

| Destination | Route Type | Tunnel Next Hop |
|---|---|---|
| `10.10.0.0/24` | OSPF Inter-Area | `203.0.113.10` |
| `10.10.1.0/30` | OSPF | `203.0.113.10` |
| `10.10.1.4/30` | OSPF Inter-Area | `203.0.113.10` |
| `10.99.0.0/22` | OSPF External Type 2 | `203.0.113.10` |

The corresponding routing output showed the default route through WAN2:

```text
S* 0.0.0.0/0 [20/0] via 203.0.113.1, port2
```

### 5.3 Routing Observation

The captured routing tables show the remote networks associated with both `Cairo-Alex-WAN1` and `Cairo-Alex-WAN2`. They also show the default route changing between WAN1 and WAN2 in the recorded Dual-WAN tests.

**Result: PASS.** The routing evidence demonstrates route selection through the recorded WAN paths and VPN tunnel interfaces.

These route snapshots are evidence of routing state at their respective capture times. They do not, by themselves, establish uninterrupted application sessions during a transition.

## 6. Additional Host Routes

The available routing evidence includes the following `/32` routes:

- `172.16.0.1/32`
- `172.16.0.2/32`
- `172.16.0.5/32`
- `172.16.0.6/32`

The captured FortiGate output shows these addresses associated with the Cairo–Alexandria tunnel interfaces, including connected tunnel addresses and routes learned through the alternate tunnel.

Their presence is recorded as observed. Any change to these routes should be based on the full routing configuration and the intended tunnel design.

## 7. Verification Scope

The collected evidence confirms the following:

- OSPF adjacencies on `HQ-CORE-01` were in the `FULL` state during the recorded check.
- A static default route toward the FortiGate transit connection was present on the core.
- The listed remote and connected routes appeared in the core routing table.
- An OSPF External Type 2 route for `10.99.0.0/22` was present.
- FortiGate routing tables showed the recorded remote networks through both Cairo–Alexandria tunnel interfaces.
- The FortiGate default route was observed through WAN1 and through WAN2 in separate test states.

The following are separate from these routing-table results:

- Application-level connectivity between all HQ and Alexandria networks.
- Uninterrupted application sessions during route transitions.
- External HTTPS access to the DMZ through both WAN VIPs.

Dual-WAN failover and recovery results are documented in [Dual-WAN Verification](dual-wan-verification.md). VPN implementation and verification are documented in [IPsec VPN Verification](ipsec-vpn.md).

## 8. Conclusion

**PASS — The captured routing evidence confirms OSPF neighbor formation, the core default route, the documented remote routes, and route selection through both recorded Cairo–Alexandria VPN tunnel paths.**

The routing tables show the expected WAN1 and WAN2 default-route states in their respective captures. These results support the documented routing design and are consistent with the recorded Dual-WAN failover and recovery behavior.

Routing-table evidence is not presented as proof of every application flow. The separate DMZ WAN2 HTTPS publishing issue remains documented in the firewall policy verification record and does not change the results recorded here.
