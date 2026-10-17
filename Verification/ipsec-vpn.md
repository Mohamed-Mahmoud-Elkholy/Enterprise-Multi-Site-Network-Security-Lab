# IPsec VPN Verification

## 1. Overview

This document records the observed verification status of the site-to-site IPsec VPN connectivity between Cairo Headquarters and Alexandria Branch.

The design includes two route-based IPsec VPN tunnels:

| Tunnel | Purpose | Configured Preference |
|---|---|---|
| Tunnel A | Primary inter-site path | Preferred |
| Tunnel B | Backup inter-site path | Secondary |

The configured preference describes the intended path selection. Successful automatic failover and recovery must be demonstrated through separate tests.

## 2. Observed Evidence

The collected FortiGate command outputs showed:

- Active IPsec Security Associations for both configured tunnels.
- Encapsulation and decapsulation packet counters.
- OSPF neighbor relationships over both displayed tunnel interfaces on `FG-ALEX-01`.
- OSPF-learned routes associated with the inter-site tunnel endpoint addresses.

These observations indicate IPsec tunnel activity and routing adjacency at the time the commands were collected.

They do not independently establish that application traffic can traverse both tunnels or that the backup tunnel takes over successfully when the preferred path becomes unavailable.

## 3. Verification Results

| Verification Item | Observed Status | Interpretation |
|---|---|---|
| IPsec Security Associations | Observed on both configured tunnels | Tunnel activity was present during capture |
| Encapsulation and decapsulation counters | Present in collected output | Packet counters indicate observed tunnel traffic; they do not alone prove successful application connectivity |
| OSPF neighbors over tunnel interfaces | Displayed as established on `FG-ALEX-01` | Routing adjacencies were present at the time of capture |
| OSPF-learned tunnel-related routes | Present in collected output | Relevant routes were available in the displayed routing information |
| End-to-end connectivity between sites | Not confirmed | Requires explicit reachability and application tests |
| Automatic tunnel failover | Not confirmed | Requires a controlled primary-path failure test |
| Recovery to the preferred tunnel | Not confirmed | Requires a recovery test and routing evidence |

## 4. Tests Still Required

The following tests remain open until supporting results are collected:

### 4.1 End-to-End Connectivity

Verify reachability between appropriate source and destination networks at Cairo HQ and Alexandria Branch.

Where possible, test both basic network reachability and an application-level connection. Record the source, destination, test method, and result.

### 4.2 Application Connectivity Across Both Tunnels

Verify that the intended application traffic can traverse the primary and backup paths.

Tunnel status and OSPF adjacency alone are not sufficient to confirm application availability.

### 4.3 Automatic Failover

Perform a controlled test that makes the preferred tunnel unavailable.

Collect evidence showing:

- The initial preferred path and its operational state.
- The event that makes the primary path unavailable.
- The resulting routing or path change.
- Successful connectivity through the backup path, if achieved.

Do not mark failover as successful solely because the backup tunnel's Security Association remains active.

### 4.4 Recovery to the Preferred Path

Restore the preferred tunnel and verify whether traffic returns to the intended path.

Collect the relevant tunnel status, routing information, and connectivity results before marking recovery as verified.

### 4.5 Packet Loss and Service Continuity

Measure connectivity before, during, and after failover.

Record any packet loss, interruption duration, and application impact observed during the test. Do not report uninterrupted service unless the test evidence supports that conclusion.

## 5. Verification Scope and Limitations

The current evidence supports the following conclusions:

- Both configured IPsec tunnels showed Security Association activity in the collected outputs.
- Encapsulation and decapsulation counters were present.
- OSPF adjacencies were displayed over both tunnel interfaces on `FG-ALEX-01`.
- OSPF-learned routes associated with the tunnel endpoint addresses were displayed.

The current evidence does not establish:

- Successful end-to-end application connectivity between Cairo and Alexandria.
- Successful automatic failover from the primary tunnel to the backup tunnel.
- Successful recovery and return to the preferred path.
- The amount of packet loss or service interruption during a path transition.

These items must remain unverified until the relevant tests have been performed and documented.

## 6. Conclusion

The collected outputs provide evidence of IPsec Security Association activity and OSPF adjacency over the displayed tunnel interfaces.

However, successful end-to-end communication, application availability across both paths, automatic primary-to-backup failover, and recovery to the preferred path remain unverified.

The VPN design should therefore be described as showing observed tunnel and routing activity, with operational failover and service continuity still requiring validation.

Only results supported by collected test evidence should be recorded as successful.
