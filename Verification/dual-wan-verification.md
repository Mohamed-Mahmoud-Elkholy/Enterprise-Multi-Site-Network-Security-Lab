# Dual-WAN Failover and Recovery Verification

## 1. Objective

Verify automatic WAN failover from ISP-A (WAN1) to ISP-B (WAN2), followed by recovery to the preferred WAN1 route.

The lab uses WAN1 as the primary Internet path and WAN2 as the backup path. FortiGate link monitors track the reachability of their configured targets and support the primary/backup routing behavior.

## 2. WAN Configuration

| Parameter | WAN1 - ISP-A | WAN2 - ISP-B |
|---|---|---|
| FortiGate Interface | `port1` | `port2` |
| FortiGate IP | `198.51.100.2/29` | `203.0.113.2/29` |
| Gateway | `198.51.100.1` | `203.0.113.1` |
| Monitor Target | `198.51.100.250` | `203.0.113.250` |
| Default Route Distance | 10 | 20 |

## 3. Verification Results

### 3.1 Normal Operation

The routing-table evidence shows WAN1 selected as the preferred default route.

```text
S* 0.0.0.0/0 [10/0] via 198.51.100.1, port1
```

The link-monitor evidence shows both monitors in the `alive` state:

- `WAN1-MONITOR`: `alive`
- `WAN2-MONITOR`: `alive`

The recorded monitor output reports `0.000%` packet loss for both targets in this state.

**Result: PASS.** WAN1 is preferred while both monitored paths are available.

Evidence:
- `Images/Dual-WAN-Routing-Before-Failover.png`
- `Images/Dual-WAN-Monitor-Before-Failover.png`

### 3.2 WAN1 Failure and Failover to WAN2

The ISP-A connection toward INET was disconnected during the lab test.

The link-monitor output showed:

- `WAN1-MONITOR`: `dead`
- `WAN2-MONITOR`: `alive`
- WAN1 monitor packet loss: `54.000%` in the captured failure state.

The routing table changed to the WAN2 default route:

```text
S* 0.0.0.0/0 [20/0] via 203.0.113.1, port2
```

The FortiGate ping test to the WAN2 monitor target `203.0.113.250` succeeded:

```text
5 packets transmitted, 5 packets received, 0% packet loss
```

**Result: PASS.** The recorded test demonstrates failover from WAN1 to WAN2, with WAN1 detected as unavailable and the WAN2 monitor target reachable.

Evidence:
- `Images/Dual-WAN-Monitor-During-Failover.png`
- `Images/Dual-WAN-Failover-to-WAN2.png`
- `Images/Dual-WAN-Ping-WAN2-Success.png`

### 3.3 WAN1 Recovery

After the ISP-A connection was restored, the routing table returned to WAN1:

```text
S* 0.0.0.0/0 [10/0] via 198.51.100.1, port1
```

The recovered link-monitor output showed:

- `WAN1-MONITOR`: `alive`
- `WAN2-MONITOR`: `alive`
- WAN1 monitor packet loss: `0.000%`
- WAN2 monitor packet loss: `0.000%`

The FortiGate ping test to the WAN1 monitor target `198.51.100.250` also succeeded:

```text
5 packets transmitted, 5 packets received, 0% packet loss
```

**Result: PASS.** The preferred default route returned to WAN1, both monitors were alive in the captured recovery state, and both monitor targets reported zero packet loss in that output.

Evidence:
- `Images/Dual-WAN-Recovery-to-WAN1.png`
- `Images/Dual-WAN-Ping-WAN1-Recovery.png`

## 4. HA Cluster Context

The accompanying HA status screenshot shows:

- HA mode: `A-P`
- Cluster health: `OK`
- Primary member: `FG-HQ-1`
- Secondary member: `FG-HQ-2`
- Both members: `in-sync`

This records the cluster state shown in the evidence. It is separate from the WAN failover test and does not, by itself, prove uninterrupted traffic during a FortiGate HA member failure.

Evidence:
- `Images/Dual-WAN-HA-Before-Failover.png`

## 5. Observations

- WAN1 was the preferred default route during normal operation.
- When the ISP-A path was disconnected, `WAN1-MONITOR` became `dead`.
- The default route switched to WAN2 through `203.0.113.1` on `port2`.
- The WAN2 monitor target responded successfully to ping during the failover test.
- After ISP-A was restored, `WAN1-MONITOR` returned to `alive`.
- The preferred default route returned to WAN1 through `198.51.100.1` on `port1`.
- The recovered monitor output reported `0.000%` packet loss for both WAN targets.
- The `54.000%` WAN1 packet loss shown in the failure-state screenshot corresponds to the captured WAN1 outage condition; the recovered-state screenshot subsequently shows zero packet loss.
- The evidence demonstrates default-route failover and recovery. It does not independently establish uninterrupted application sessions or zero packet loss throughout the transition.

## 6. Evidence Inventory

| Evidence File | What It Documents |
|---|---|
| `Dual-WAN-HA-Before-Failover.png` | FortiGate HA cluster status |
| `Dual-WAN-Routing-Before-Failover.png` | WAN1 preferred default route |
| `Dual-WAN-Monitor-Before-Failover.png` | Both WAN monitors alive |
| `Dual-WAN-Monitor-During-Failover.png` | WAN1 dead and WAN2 alive during failure |
| `Dual-WAN-Failover-to-WAN2.png` | Default route switched to WAN2 |
| `Dual-WAN-Ping-WAN2-Success.png` | Successful ping to the WAN2 monitor target |
| `Dual-WAN-Recovery-to-WAN1.png` | Default route returned to WAN1 |
| `Dual-WAN-Ping-WAN1-Recovery.png` | Successful ping to the WAN1 monitor target after recovery |

## 7. Overall Result

**PASS — Dual-WAN default-route failover and recovery were demonstrated in the recorded lab tests.**

The captured evidence confirms that the default route switched from WAN1 to WAN2 during the WAN1 failure and returned to WAN1 after recovery. Both monitor targets were reachable in their respective recorded tests, and the recovered-state monitor output showed zero packet loss for both paths.

This result confirms the demonstrated primary/backup routing behavior. It does not claim that application sessions remained uninterrupted throughout the transition or that every possible Internet failure scenario has been tested.
