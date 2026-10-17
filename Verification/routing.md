# Routing Verification

## 1. Test Environment

- **Device:** `HQ-CORE-01`
- **Routing protocol:** OSPF
- **Verification commands:**
  - `show ip ospf neighbor`
  - `show ip route`

## 2. OSPF Neighbor Verification

The command `show ip ospf neighbor` showed the following:

- Neighbor ID `3.3.3.3` on `Vlan901`, address `192.168.0.130`, state `FULL/DR`.
- Neighbor ID `2.2.2.2` appeared on the local VLAN interfaces, with the displayed neighbor state `FULL/DR`.

**Result:** The displayed OSPF adjacencies were in the FULL state when the command was executed.

## 3. Routing Table Verification

The command `show ip route` showed the following routing entries.

### Default Route

```text
S* 0.0.0.0/0 via 192.168.0.130
```

The default route points to `192.168.0.130` through the FortiGate transit connection.

### Remote Routes

| Destination | Route Type | Next Hop | Interface |
|---|---|---|---|
| `10.10.0.0/24` | OSPF Inter-Area | `192.168.0.130` | `Vlan901` |
| `10.10.1.0/30` | OSPF | `192.168.0.130` | `Vlan901` |
| `10.10.1.4/30` | OSPF Inter-Area | `192.168.0.130` | `Vlan901` |
| `10.99.0.0/22` | OSPF External Type 2 | `192.168.0.130` | `Vlan901` |
| `192.168.0.136/30` | OSPF | `192.168.0.130` | `Vlan901` |
| `192.168.0.144/28` | OSPF | `192.168.0.130` | `Vlan901` |

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

## 4. OSPF Route Redistribution Evidence

The routing table on `HQ-CORE-01` displayed the following external route:

- **Destination:** `10.99.0.0/22`
- **Route type:** OSPF External Type 2
- **Next hop:** `192.168.0.130`
- **Interface:** `Vlan901`

This entry confirms that an OSPF External Type 2 route was present in the routing table at the time of verification.

The routing table entry alone does not identify the source protocol or prove the complete redistribution configuration. The relevant routing configuration should be documented separately to confirm those details.

## 5. Verification Scope

The available command outputs confirm the displayed OSPF neighbor states, the default route toward the FortiGate transit connection, and the listed routing entries on `HQ-CORE-01`.

These results provide evidence of the routing information available on the core switch at the time of testing. They do not, by themselves, prove end-to-end connectivity, Internet availability, VPN failover, or DMZ service availability.

## 6. Items Requiring Further Verification

The following items require additional evidence:

- Verify end-to-end connectivity between Cairo HQ and the Alexandria branch.
- Verify Internet access and dual-WAN failover separately.
- Investigate why the displayed `172.16.0.1`, `172.16.0.2`, `172.16.0.5`, and `172.16.0.6` routes appear as `/32` host routes.
- Verify the routing and operational status of the DMZ and `WEB01`.
- Collect corresponding routing evidence from the other relevant devices.
- Review the routing configuration to document the source protocol and configuration used for OSPF route redistribution.
