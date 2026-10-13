# FortiGate HA Verification

## Overview

The Cairo Headquarters lab uses two FortiGate VM64-KVM appliances configured in Active-Passive (A-P) High Availability mode.

## Cluster Members

| Parameter | FG-HQ-1 | FG-HQ-2 |
|---|---|---|
| HA Role | Primary | Secondary |
| Cluster Index | 0 | 1 |
| Configuration Status | In-sync | In-sync |
| Heartbeat Interfaces | port3, port4 | port3, port4 |
| Monitored Interfaces | port1, port2, port5, port6 | port1, port2, port5, port6 |
| Dedicated Management Interface | port8 | port8 |

## Current Verification Results

The following results were observed during verification:

- HA health status reported `OK`.
- Both cluster members were visible in the HA status output.
- `FG-HQ-1` was Primary and `FG-HQ-2` was Secondary.
- Both members reported `in-sync`.
- Heartbeat interfaces `port3` and `port4` were up on both appliances.
- Both heartbeat interfaces reported zero RX drops, RX errors, and TX errors in the collected interface statistics.
- Monitored interfaces `port1`, `port2`, `port5`, and `port6` were up in the collected HA status output.

## Historical HA Events

The HA history contains previous events in which `FG-HQ-2` lost heartbeat connectivity on `port3` and `port4`. The primary appliance subsequently operated as the only cluster member during some of these events.

The available evidence does not establish the root cause of these historical events.

Although the heartbeat links and their interface counters looked healthy during the latest checks, this does not prove that the historical interruptions have been permanently resolved.

## Failover Validation

**Status: Not yet formally validated.**

The current evidence confirms the cluster's observed health, member roles, configuration synchronization, and heartbeat interface status.

It does not establish that production traffic successfully transfers to `FG-HQ-2` during a controlled failure of `FG-HQ-1`.

A controlled failover test must be completed before operational HA failover can be marked as verified.

## Evidence

- `Images/FG-HQ-HA-Status-Primary.png`
- `Images/FG-HQ-HA-Status-Secondary.png`
- `Images/FG-HQ-HA-Status-In-Sync.png`

Ensure that the filenames above match the actual files committed to the repository.

## Conclusion

The FortiGate Active-Passive cluster was observed in a healthy, synchronized state. Both heartbeat interfaces were up on both appliances during verification. Historical heartbeat interruptions remain unexplained, and successful controlled failover is still unverified.
