# Dual-WAN Verification

## Overview

The Cairo Headquarters lab is designed with two simulated ISP connections to provide redundant Internet connectivity.

## WAN Addressing

| Parameter | WAN1 | WAN2 |
|---|---|---|
| FortiGate Interface | `port1` | `port2` |
| ISP Gateway | `198.51.100.1` | `203.0.113.1` |
| FortiGate IP | `198.51.100.2/29` | `203.0.113.2/29` |
| Monitoring Target | `198.51.100.250` | `203.0.113.250` |

These addresses describe the planned lab addressing. The current interface configuration and connectivity must be checked against the running devices.

## Verification Requirements

The following tests are required before dual-WAN redundancy can be considered validated:

- Verify that each FortiGate WAN interface can reach its ISP gateway.
- Verify reachability of the configured monitoring target through the correct WAN path.
- Confirm the active default route under normal conditions.
- Disconnect or disable the primary WAN in a controlled test.
- Confirm that Internet connectivity recovers through the backup WAN.
- Restore the primary WAN and verify route recovery.
- Record the routing table and connectivity test results for each state.

## Current Status

Dual-WAN failover is **not confirmed by this document**. Successful failover must be supported by collected routing and connectivity evidence.

## Conclusion

The lab includes a dual-WAN design intended to provide Internet-path redundancy. Operational failover remains subject to testing and verification.
