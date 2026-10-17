# Dual-WAN Failover and Recovery Verification

## 1. Objective

Verify automatic WAN failover from ISP-A (WAN1) to ISP-B (WAN2), followed by recovery to the preferred WAN1 route.

The lab uses WAN1 as the primary Internet path and WAN2 as the backup path. Link monitors track the reachability of their configured targets and support the intended primary/backup routing behavior.

## 2. WAN Configuration

| Parameter | WAN1 - ISP-A | WAN2 - ISP-B |
|---|---|---|
| FortiGate Interface | `port1` | `port2` |
| FortiGate IP | `198.51.100.2/29` | `203.0.113.2/29` |
| Gateway | `198.51.100.1` | `203.0.113.1` |
| Monitor Target | `198.51.100.250` | `203.0.113.250` |
| Default Route Distance | 10 | 20 |

## 3. Test Results

### 3.1 Normal Operation

Collected lab evidence showed both WAN paths available and WAN1 selected as the preferred default route.

Observed route:

```text
S* 0.0.0.0/0 [10/0] via 198.51.100.1, port1
```

**Result: PASS.** WAN1 was selected as the preferred default route under normal conditions.

### 3.2 WAN1 Failure and Failover to WAN2

The ISP-A connection toward INET was disconnected in the lab to test failover.

Observed results:

- `WAN1-MONITOR` reported `dead`.
- `WAN2-MONITOR` reported `alive`.
- The preferred default route switched to WAN2.

Observed route:

```text
S* 0.0.0.0/0 [20/0] via 203.0.113.1, port2
```

Ping to the WAN2 monitor target `203.0.113.250` succeeded.

**Result: PASS.** The lab demonstrated default-route failover from WAN1 to WAN2, and the WAN2 monitor target was reachable during the recorded test.

### 3.3 WAN1 Recovery

The ISP-A connection toward INET was restored.

Observed results:

- `WAN1-MONITOR` returned to `alive`.
- `WAN2-MONITOR` remained `alive`.
- The preferred default route returned to WAN1.

Observed route:

```text
S* 0.0.0.0/0 [10/0] via 198.51.100.1, port1
```

Ping to the WAN1 monitor target `198.51.100.250` succeeded after recovery.

**Result: PASS.** The preferred default route returned to WAN1 after its recovery, and the WAN1 monitor target responded to ping.

### 3.4 WAN1 Monitor Stability After Recovery

After WAN1 recovered, the Link Monitor output displayed **63% packet loss**.

This result is recorded as a specific stability observation. The successful ping test and restored preferred route confirm recovery behavior in the captured test, while the reported packet loss indicates that monitor stability was not consistently demonstrated by that output.

The available evidence does not establish whether the packet loss was temporary or persistent.

**Result: RECOVERY OBSERVED — MONITOR STABILITY REQUIRES FOLLOW-UP.**

## 4. Evidence

The following screenshots document the recorded tests:

- [WAN1 Monitor During Failover](../Images/Dual-WAN-Monitor-During-Failover.png)
- [Default Route Switched to WAN2](../Images/Dual-WAN-Failover-to-WAN2.png)
- [WAN2 Monitor Target Ping Success](../Images/Dual-WAN-Ping-WAN2-Success.png)
- [Default Route Recovered to WAN1](../Images/Dual-WAN-Recovery-to-WAN1.png)
- [WAN1 Monitor Target Ping After Recovery](../Images/Dual-WAN-Ping-WAN1-Recovery.png)

## 5. Observations

- WAN1 was selected as the preferred default route during normal operation.
- When the ISP-A connection toward INET was disconnected, the WAN1 monitor reported `dead` and the default route switched to WAN2.
- The WAN2 monitor remained `alive`, and ping to `203.0.113.250` succeeded during the recorded failover test.
- After the ISP-A connection was restored, the WAN1 monitor returned to `alive` and the preferred default route returned to WAN1.
- Ping to `198.51.100.250` succeeded after recovery.
- The Link Monitor output subsequently displayed 63% packet loss, so WAN1 monitor stability after recovery needs a focused follow-up check.
- The captured route changes demonstrate the intended primary/backup default-route selection behavior.
- These tests do not independently establish uninterrupted application sessions or zero packet loss throughout the transition.

## 6. Follow-Up Check

The only follow-up item recorded in this document is WAN1 monitor stability after recovery.

To close that item, repeat the monitor observation after recovery and compare the reported packet loss with reachability to the WAN1 monitor target. Record the result and supporting screenshot.

No change to the existing failover configuration should be made solely because one monitor output displayed 63% packet loss; first establish whether the behavior is reproducible.

## 7. Overall Result

**PASS — WAN1-to-WAN2 default-route failover and recovery to WAN1 were observed in the lab.**

The primary/backup route-selection behavior worked as expected in the recorded tests. WAN1 monitor stability after recovery remains a specific follow-up item because the captured Link Monitor output displayed 63% packet loss.

This result does not classify the entire Dual-WAN implementation as incomplete. It distinguishes the demonstrated failover and recovery behavior from the remaining monitor-stability observation.
