# FortiGate HA Verification

## 1. Overview

The Cairo Headquarters lab uses two FortiGate VM64-KVM appliances configured in Active-Passive (A-P) High Availability mode.

The purpose of this document is to record the observed cluster state, synchronization status, heartbeat interfaces, role changes, recovery evidence, and the limitations of the available failover tests.

The evidence distinguishes between a healthy synchronized cluster and independently verified operational failover.

## 2. Cluster Members

| Parameter | FG-HQ-1 | FG-HQ-2 |
|---|---|---|
| Normal HA Role | Primary | Secondary |
| Cluster Index | 0 | 1 |
| Configuration Status | In-sync | In-sync |
| Heartbeat Interfaces | port3, port4 | port3, port4 |
| Monitored Interfaces | port1, port2, port5, port6 | port1, port2, port5, port6 |
| Dedicated Management Interface | port8 | port8 |

These values describe the recorded normal cluster state. The active Primary role may change during an HA event.

## 3. Normal Cluster State Verification

The collected HA status output showed the following:

- HA health status reported `OK`.
- Both cluster members were visible in the HA status output.
- `FG-HQ-1` was Primary and `FG-HQ-2` was Secondary during the recorded normal state.
- Both members reported `in-sync`.
- Heartbeat interfaces `port3` and `port4` were up on both appliances in the collected status output.
- The collected heartbeat interface statistics reported zero RX drops, RX errors, and TX errors at the time of that check.
- Monitored interfaces `port1`, `port2`, `port5`, and `port6` were up in the collected HA status output.

**Result: PASS — A healthy, synchronized cluster state was observed during the recorded check.**

This result does not establish uninterrupted traffic during a failover event.

### 3.1 Normal Cluster Evidence

**HA cluster synchronization**

![HA Cluster In-Sync](../Images/FG-HQ-HA-Status-In-Sync.png)

**FG-HQ-1 Primary**

![FG-HQ-1 Primary](../Images/FG-HQ-HA-Status-Primary.png)

**FG-HQ-2 Secondary**

![FG-HQ-2 Secondary](../Images/FG-HQ-HA-Status-Secondary.png)

## 4. Historical HA Heartbeat Events

The HA history contains previous events in which `FG-HQ-2` lost heartbeat connectivity on `port3` and `port4`. During some of these events, the primary appliance subsequently operated as the only cluster member.

The available evidence does not establish the root cause of these historical events.

Although the heartbeat links and their interface counters looked healthy during the recorded checks, this does not prove that the historical interruptions have been permanently resolved.

**Status: UNRESOLVED ROOT CAUSE — Historical heartbeat interruptions require separate investigation if they recur.**

Potential troubleshooting evidence includes the HA event history, heartbeat interface status, interface error counters, and synchronized timestamps from both cluster members. No specific root cause should be claimed without supporting evidence.

## 5. Observed HA Role Change and Recovery

Additional screenshots show a change in the displayed HA roles, with `FG-HQ-2` appearing as Primary and `FG-HQ-1` as Secondary. A separate screenshot records an HA recovery state.

### 5.1 Observed Role Change

![Observed HA Role Change](../Images/FG-HQ-HA-Failover-to-HQ-2.png)

### 5.2 HA Recovery

![HA Recovery](../Images/FG-HQ-HA-Recovery.png)

**Result: PARTIAL — A role change and a recovery state were captured.**

These screenshots provide evidence of the displayed role transition and recovery state. They do not, by themselves, establish that a controlled failure of `FG-HQ-1` was performed or that production traffic continued without interruption during the transition.

## 6. Failover Validation

**Status: Partially documented; end-to-end HA failover validation remains unconfirmed.**

The available evidence supports these conclusions:

- The cluster was observed in a healthy, synchronized state.
- Both members and their HA roles were visible in the collected status output.
- A change in the displayed Primary/Secondary roles was observed.
- A recovery state was captured.

The available screenshots do not conclusively demonstrate successful traffic continuity during a controlled failover.

### 6.1 Evidence Required for Full Operational Validation

To mark operational HA failover as fully verified, capture a controlled test containing:

1. **Baseline:** Record the current Primary and Secondary roles and confirm that both members are synchronized.
2. **Pre-failure connectivity:** Run a continuous ping or another suitable connectivity test through the HA cluster and record the baseline.
3. **Controlled trigger:** Record the specific failure introduced, such as shutting down the active member or disconnecting a monitored interface. State exactly what was changed.
4. **Role transition:** Capture the HA status output showing the new Primary and Secondary roles.
5. **Traffic during transition:** Preserve the continuous test output, including any packet loss or interruption.
6. **Post-failover connectivity:** Verify that the intended destination remains reachable through the new Primary.
7. **Recovery:** Restore the original condition and capture the resulting HA roles, synchronization state, and connectivity.

Report the measured packet loss and interruption duration if they were captured. Do not claim zero packet loss or seamless failover without evidence.

These steps describe the evidence needed for a future validation test; they are not a claim that the test has already been performed.

## 7. Evidence Files

The following screenshots are referenced in this document:

| Evidence File | Purpose |
|---|---|
| `Images/FG-HQ-HA-Status-In-Sync.png` | Cluster synchronization status |
| `Images/FG-HQ-HA-Status-Primary.png` | Normal Primary role |
| `Images/FG-HQ-HA-Status-Secondary.png` | Normal Secondary role |
| `Images/FG-HQ-HA-Failover-to-HQ-2.png` | Observed role change |
| `Images/FG-HQ-HA-Recovery.png` | Recorded recovery state |

Ensure these filenames match the files committed to the repository. The referenced screenshots are evidence of the displayed states, not a substitute for a controlled traffic-continuity test.

## 8. Conclusion

The FortiGate Active-Passive cluster was observed in a healthy, synchronized state, with both heartbeat interfaces up during the recorded checks. Additional screenshots show an HA role change and a recovery state.

Historical heartbeat interruptions remain unexplained. Successful traffic continuity during a controlled HA failover has not yet been conclusively verified, so operational failover should not be marked as fully validated.

The next verification priority is to capture one controlled HA failover test with a documented trigger, role transition, traffic results, and recovery evidence.
