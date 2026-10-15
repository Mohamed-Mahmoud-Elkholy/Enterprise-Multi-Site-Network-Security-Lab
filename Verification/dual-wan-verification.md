# Dual-WAN Failover and Recovery Verification

## Objective

Verify automatic WAN failover from ISP-A (WAN1) to ISP-B (WAN2), followed by recovery to the preferred WAN1 route.

## WAN Configuration

| Parameter | WAN1 - ISP-A | WAN2 - ISP-B |
|---|---|---|
| FortiGate Interface | `port1` | `port2` |
| FortiGate IP | `198.51.100.2/29` | `203.0.113.2/29` |
| Gateway | `198.51.100.1` | `203.0.113.1` |
| Monitor Target | `198.51.100.250` | `203.0.113.250` |
| Default Route Distance | 10 | 20 |

## Test Results

### 1. Normal Operation

- WAN1 and WAN2 Link Monitors were both reported as `alive`.
- The preferred default route used WAN1.

```text
S* 0.0.0.0/0 [10/0] via 198.51.100.1, port1
```

### 2. WAN1 Failure and Failover

The ISP-A connection toward INET was disconnected in the lab.

- `WAN1-MONITOR`: `dead`
- `WAN2-MONITOR`: `alive`
- The default route switched to WAN2.

```text
S* 0.0.0.0/0 [20/0] via 203.0.113.1, port2
```

- Ping to the WAN2 monitor target `203.0.113.250` succeeded.

### 3. WAN1 Recovery

The ISP-A connection toward INET was restored.

- `WAN1-MONITOR` returned to `alive`.
- `WAN2-MONITOR` remained `alive`.
- The preferred default route returned to WAN1.

```text
S* 0.0.0.0/0 [10/0] via 198.51.100.1, port1
```

- Ping to the WAN1 monitor target `198.51.100.250` succeeded after recovery.

## Evidence

- [WAN1 Monitor During Failover](../Images/Dual-WAN-Monitor-During-Failover.png)
- [Default Route Switched to WAN2](../Images/Dual-WAN-Failover-to-WAN2.png)
- [WAN2 Monitor Target Ping Success](../Images/Dual-WAN-Ping-WAN2-Success.png)
- [Default Route Recovered to WAN1](../Images/Dual-WAN-Recovery-to-WAN1.png)
- [WAN1 Monitor Target Ping After Recovery](../Images/Dual-WAN-Ping-WAN1-Recovery.png)

## Observations and Limitations

- Automatic default-route failover and recovery were observed in the lab.
- Ping tests confirmed reachability to the configured monitor targets after failover and recovery.
- These tests do not, by themselves, prove uninterrupted application sessions or end-to-end Internet availability during the transition.
- After WAN1 recovery, the Link Monitor output showed `alive` but reported 63% packet loss in its displayed statistics. A follow-up stability check is required before declaring WAN1 fully stable.

## Overall Result

**PASS — Default-route failover to WAN2 and recovery to WAN1 were observed.**

Application continuity and WAN1 stability after recovery remain outside the scope of this completed test.
