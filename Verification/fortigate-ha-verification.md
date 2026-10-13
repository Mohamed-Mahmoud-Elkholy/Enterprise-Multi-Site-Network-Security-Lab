# FortiGate HA Verification

## Overview

The Cairo Headquarters security infrastructure uses two FortiGate VM64-KVM appliances configured in Active-Passive High Availability (HA) mode.

## Cluster Members

| Parameter | FG-HQ-1 | FG-HQ-2 |
|---|---|---|
| HA Role | Primary | Secondary |
| Cluster Index | 0 | 1 |
| Configuration Status | In-sync | In-sync |
| HA Heartbeat Interfaces | port3, port4 | port3, port4 |

## Verification Results

The following results were observed during verification:

- HA health status reported `OK`.
- Both cluster members were visible in the HA status output.
- `FG-HQ-1` was the current Primary.
- `FG-HQ-2` was the current Secondary.
- Both members reported `in-sync`.
- Heartbeat interfaces `port3` and `port4` were reported as up on both members.
- Monitored interfaces `port1`, `port2`, `port5`, and `port6` were reported as up on both members.

## Failover Validation

**Status: Not yet tested.**

The collected output confirms the current HA cluster status and configuration synchronization. It does not prove that traffic successfully transfers to the Secondary during a Primary failure.

A controlled failover test is required before operational HA failover can be considered verified.

## Evidence

- `Images/FG-HQ-HA-Status-Primary.png`
- `Images/FG-HQ-HA-Status-Secondary.png`
- `Images/FG-HQ-HA-Status-In-Sync.png`

## Conclusion

The FortiGate HA cluster was observed in a healthy state with both members synchronized. Actual failover behavior remains unverified pending a controlled test.
