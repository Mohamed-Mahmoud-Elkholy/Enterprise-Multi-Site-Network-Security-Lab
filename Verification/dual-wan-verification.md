# Dual-WAN Failover and Recovery Verification

## 1. Objective

Verify automatic WAN failover from ISP-A (WAN1) to ISP-B (WAN2), followed by recovery to the preferred WAN1 route.

The lab uses WAN1 as the primary Internet path and WAN2 as the backup path. FortiGate link monitors track the reachability of their configured targets.

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

The captured routing-table evidence showed WAN1 selected as the preferred default route.

```text
S* 0.0.0.0/0 [10/0] via 198.51.100.1, port1
```

Both monitors were alive, and the recorded monitor output showed zero packet loss for both targets.

**Result: PASS — WAN1 was preferred while both monitored paths were available.**

**Evidence — WAN1 preferred route**

![WAN1 preferred default route](../Images/Dual-WAN-Routing-Before-Failover.png)

**Evidence — Both WAN monitors alive**

![WAN monitors before failover](../Images/Dual-WAN-Monitor-Before-Failover.png)

### 3.2 WAN1 Failure and Failover to WAN2

The ISP-A connection toward INET was disconnected during the lab test.

The captured results showed:

- `WAN1-MONITOR`: `dead`
- `WAN2-MONITOR`: `alive`
- WAN1 monitor packet loss: `54.000%` in the captured failure state.

The routing table changed to WAN2:

```text
S* 0.0.0.0/0 [20/0] via 203.0.113.1, port2
```

The FortiGate ping test to `203.0.113.250` succeeded:

```text
5 packets transmitted, 5 packets received, 0% packet loss
```

**Result: PASS — Failover from WAN1 to WAN2 was demonstrated.**

**Evidence — Monitor state during failover**

![WAN1 down and WAN2 alive](../Images/Dual-WAN-Monitor-During-Failover.png)

**Evidence — Default route switched to WAN2**

![Failover to WAN2](../Images/Dual-WAN-Failover-to-WAN2.png)

**Evidence — WAN2 monitor target ping succeeded**

![WAN2 ping success](../Images/Dual-WAN-Ping-WAN2-Success.png)

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

The ping test to `198.51.100.250` also succeeded:

```text
5 packets transmitted, 5 packets received, 0% packet loss
```

**Result: PASS — The preferred route returned to WAN1, and both monitor targets showed zero packet loss in the captured recovery state.**

**Evidence — Default route recovered to WAN1**

![Recovery to WAN1](../Images/Dual-WAN-Recovery-to-WAN1.png)

**Evidence — WAN1 monitor target ping succeeded after recovery**

![WAN1 ping after recovery](../Images/Dual-WAN-Ping-WAN1-Recovery.png)

### 3.4 HA Cluster Context

The captured HA status showed:

- HA mode: `A-P`
- Cluster health: `OK`
- Primary member: `FG-HQ-1`
- Secondary member: `FG-HQ-2`
- Both members: `in-sync`

This evidence records the cluster state separately from the WAN failover test. It does not independently establish uninterrupted traffic during an HA member failure.

![FortiGate HA status before failover](../Images/Dual-WAN-HA-Before-Failover.png)

## 4. Observations

- WAN1 was the preferred default route during normal operation.
- Disconnecting ISP-A caused `WAN1-MONITOR` to become `dead`.
- The default route switched to WAN2 through `203.0.113.1` on `port2`.
- The WAN2 monitor target responded successfully during failover.
- After ISP-A was restored, `WAN1-MONITOR` returned to `alive`.
- The preferred default route returned to WAN1 through `198.51.100.1` on `port1`.
- The recovered monitor output reported zero packet loss for both targets.
- The `54.000%` packet loss shown during the failure corresponds to the captured WAN1 outage state; the recovery evidence subsequently shows zero packet loss.

The evidence demonstrates default-route failover and recovery. It does not independently establish uninterrupted application sessions throughout the transition or successful access to every external application through both WANs.

## 5. Evidence Inventory

| Evidence | What It Documents |
|---|---|
| [HA Cluster Before Failover](../Images/Dual-WAN-HA-Before-Failover.png) | FortiGate HA cluster status |
| [WAN1 Preferred Route](../Images/Dual-WAN-Routing-Before-Failover.png) | WAN1 selected as the preferred default route |
| [WAN Monitors Before Failover](../Images/Dual-WAN-Monitor-Before-Failover.png) | Both WAN monitors alive |
| [WAN Monitors During Failover](../Images/Dual-WAN-Monitor-During-Failover.png) | WAN1 dead and WAN2 alive |
| [Failover to WAN2](../Images/Dual-WAN-Failover-to-WAN2.png) | Default route switched to WAN2 |
| [WAN2 Ping Success](../Images/Dual-WAN-Ping-WAN2-Success.png) | Successful ping to WAN2 monitor target |
| [Recovery to WAN1](../Images/Dual-WAN-Recovery-to-WAN1.png) | Default route returned to WAN1 |
| [WAN1 Ping After Recovery](../Images/Dual-WAN-Ping-WAN1-Recovery.png) | Successful ping after recovery |

## 6. Overall Result

**PASS — Dual-WAN default-route failover and recovery were demonstrated in the recorded lab tests.**

The captured evidence shows the default route switching from WAN1 to WAN2 during the WAN1 failure and returning to WAN1 after recovery. Both monitor targets were reachable in their recorded tests, and the recovered-state monitor output showed zero packet loss for both paths.

This result confirms the demonstrated primary/backup routing behavior. It does not claim that application sessions remained uninterrupted throughout the transition or that every possible Internet failure scenario has been tested.
