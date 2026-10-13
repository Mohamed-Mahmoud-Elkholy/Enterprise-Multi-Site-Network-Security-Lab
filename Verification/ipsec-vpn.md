# IPsec VPN Verification

## Overview

This document records the observed verification status of the site-to-site IPsec VPN connectivity between Cairo Headquarters and Alexandria Branch.

The design includes two route-based IPsec VPN tunnels:

| Tunnel | Purpose | Configured Preference |
|---|---|---|
| Tunnel A | Primary inter-site path | Preferred |
| Tunnel B | Backup inter-site path | Secondary |

## Observed Evidence

The collected FortiGate command outputs showed:

- Active IPsec Security Associations for both configured tunnels.
- Encapsulation and decapsulation packet counters.
- OSPF neighbor relationships over both displayed tunnel interfaces on `FG-ALEX-01`.
- OSPF-learned routes associated with the inter-site tunnel endpoint addresses.

These observations indicate tunnel activity and routing adjacency at the time the commands were collected.

## Tests Still Required

The following tests are not considered confirmed until their results are collected:

- End-to-end connectivity between Cairo and Alexandria.
- Application connectivity across both tunnels.
- Automatic failover when the primary tunnel becomes unavailable.
- Recovery and return to the preferred path.
- Packet loss and connectivity during failover.

## Conclusion

The collected outputs provide evidence of IPsec Security Association activity and OSPF adjacency over the displayed tunnel interfaces.

Successful end-to-end communication and automatic primary/backup failover remain unverified.

Only validated test results should be added to this document.
