# Dual-WAN Verification

## Overview

The Cairo Headquarters lab is designed with two simulated ISP connections to provide redundant Internet connectivity through the FortiGate firewall.

## WAN Addressing

| Parameter | WAN1 | WAN2 |
|---|---|---|
| FortiGate Interface | `port1` | `port2` |
| ISP Gateway | `198.51.100.1` | `203.0.113.1` |
| FortiGate IP | `198.51.100.2/29` | `203.0.113.2/29` |
| Monitoring Target | `198.51.100.250` | `203.0.113.250` |

These addresses represent the planned lab addressing. The running configuration and actual connectivity must be verified against the devices before operational status is claimed.

## Verification Requirements

The following tests are required to validate dual-WAN redundancy:

- Verify that each FortiGate WAN interface can reach its ISP gateway.
- Verify that each monitoring target is reachable through its intended WAN interface.
- Confirm the active default route under normal conditions.
- Disable the primary WAN in a controlled test.
- Confirm that the default route and Internet connectivity recover through the backup WAN.
- Restore the primary WAN and verify route recovery.
- Capture routing and connectivity evidence for each test state.

## Current Status

**Status: Not yet validated.**

The lab design includes two simulated ISP connections, but successful dual-WAN failover has not been established by the evidence documented here.

A successful ping to an internal FortiGate transit address does not prove that either WAN connection or Internet failover is working.

## Evidence Requirements

Before marking dual-WAN failover as verified, collect:

- FortiGate WAN interface status and addressing.
- Link-monitor status for both WAN connections.
- The routing table before and after the controlled failover.
- Connectivity test results before, during, and after failover.
- Confirmation that the primary WAN resumes service after restoration.

## Conclusion

The Cairo Headquarters topology is designed for Internet-path redundancy using WAN1 and WAN2. Operational failover remains unverified until the required connectivity, monitoring, and routing tests are completed and documented.
