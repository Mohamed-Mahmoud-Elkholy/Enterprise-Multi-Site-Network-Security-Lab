# Routing Verification

## 1. Test Environment

- **Device:** `HQ-CORE-01`
- **Routing protocol:** OSPF
- **Verification commands:**
  - `show ip ospf neighbor`
  - `show ip route`

This document records routing evidence collected from the running lab. Results describe the device state at the time of capture and do not automatically establish end-to-end service availability.

## 2. OSPF Neighbor Verification

The command `show ip ospf neighbor` showed the following:

- Neighbor ID `3.3.3.3` on `Vlan901`, address `192.168.0.130`, state `FULL/DR`.
- Neighbor ID `2.2.2.2` appeared on the local VLAN interfaces, with the displayed neighbor state `FULL/DR`.

**Result:** The displayed OSPF adjacencies were in the `FULL` state when the command was executed.

The collected output confirms the reported neighbor states at that time. It does not independently prove application connectivity across the routes advertised by those neighbors.

## 3. Routing Table Verification

The command `show ip route` showed the following routing entries.

### Default Route

```text
S* 0.0.0.0/0 via 192.168.0.130
```

The default route points to `192.168.0.130` through the FortiGate transit connection.

**Result:** A static default route was present in the routing table at the time of capture.

The presence of this route does not independently prove Internet connectivity or successful dual-WAN failover.

### Remote Routes

| Destination | Route Type | Next Hop | Interface |
|---|---|---|---|
| `10.10.0.0/24` | OSPF Inter-Area | `192.168.0.130` | `Vlan901` |
| `10.10.1.0/30` | OSPF | `192.168.0.130` | `Vlan901` |
| `10.10.1.4/30` | OSPF Inter-Area | `192.168.0.130` | `Vlan901` |
| `10.99.0.0/22` | OSPF External Type 2 | `192.168.0.130` | `Vlan901` |
| `192.168.0.136/30` | OSPF | `192.168.0.130` | `Vlan901` |
| `192.168.0.144/28` | OSPF | `192.168.0.130` | `Vlan901` |

These entries document the remote routes displayed by `HQ-CORE-01`. Their presence confirms that the routes were available in the local routing table at the time of capture; it does not by itself confirm that the destinations were reachable.

### Local Networks

The routing table showed the Cairo VLAN networks as directly connected:

- **VLAN 10:** `192.168.0.0/27`
- **VLAN 20:** `192.168.0.32/28`
- **VLAN 30:** `192.168.0.48/28`
- **VLAN 40:** `192.168.0.64/28`
- **VLAN 50:** `192.168.0.80/28`
- **VLAN 60:** `192.168.0.96/29`
- **VLAN 70:** `192.168.0.104/29`
- **VLAN 99:** `192.168.0.112/28`

The summary route `192.168.0.0/25` was also present with a `Null0` interface.

The connected routes document the local networks represented in the captured routing table. The `Null0` summary route is recorded as observed; its intended design purpose should be confirmed from the relevant routing configuration before drawing further conclusions.

## 4. OSPF Route Redistribution Evidence

The routing table on `HQ-CORE-01` displayed the following external route:

- **Destination:** `10.99.0.0/22`
- **Route type:** OSPF External Type 2
- **Next hop:** `192.168.0.130`
- **Interface:** `Vlan901`

**Result:** An OSPF External Type 2 route for `10.99.0.0/22` was present in the routing table at the time of verification.

The routing table entry alone does not identify the source protocol or prove the complete redistribution configuration. The relevant routing configuration must be examined separately to establish the source protocol and redistribution statements.

## 5. Additional Routes Requiring Investigation

The available notes identify the following addresses as appearing as `/32` host routes:

- `172.16.0.1/32`
- `172.16.0.2/32`
- `172.16.0.5/32`
- `172.16.0.6/32`

The reason these addresses appear as host routes has not been established by the available evidence. Their origin and intended purpose should be checked against the full routing table and the relevant device configuration before making changes.

## 6. Verification Scope

The available command outputs confirm the displayed OSPF neighbor states, the default route toward the FortiGate transit connection, and the listed routing entries on `HQ-CORE-01`.

The current evidence supports these conclusions:

- The displayed OSPF adjacencies were in the `FULL` state during the recorded check.
- A static default route toward `192.168.0.130` was present.
- The listed remote and connected routes appeared in the routing table.
- An OSPF External Type 2 route for `10.99.0.0/22` was present.

These observations do not, by themselves, prove:

- End-to-end application connectivity between Cairo HQ and Alexandria.
- Internet availability through both WAN links.
- Successful automatic dual-WAN or IPsec VPN failover and recovery.
- External HTTPS availability through the DMZ VIPs.
- The exact source protocol or complete configuration used for route redistribution.

## 7. Items Requiring Further Verification

The following items remain open:

- Verify end-to-end connectivity between Cairo HQ and the Alexandria branch using appropriate reachability and application tests.
- Verify Internet access and dual-WAN failover separately using the existing Dual-WAN verification record.
- Investigate the origin and purpose of the displayed `172.16.0.1/32`, `172.16.0.2/32`, `172.16.0.5/32`, and `172.16.0.6/32` routes.
- Verify the routing and operational status of the DMZ and `WEB01`.
- Collect and compare routing evidence from other relevant devices when required.
- Review the routing configuration to identify the source protocol and exact statements used for OSPF route redistribution.

## 8. Conclusion

The captured routing evidence shows OSPF adjacencies in the `FULL` state, a static default route toward the FortiGate transit connection, and multiple remote and connected routes on `HQ-CORE-01`.

An OSPF External Type 2 route for `10.99.0.0/22` was also present. The source protocol and exact redistribution configuration remain unconfirmed from the routing table alone.

End-to-end connectivity, complete routing redundancy, and the unexplained `/32` host routes require separate verification. No routing change should be considered a confirmed fix until the relevant behavior has been tested successfully.
