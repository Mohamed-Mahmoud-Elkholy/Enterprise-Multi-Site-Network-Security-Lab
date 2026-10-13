# VLANs and IP Addressing

## Cairo HQ — VLAN Plan

| VLAN ID | Name | Network | Subnet Mask |
|---:|---|---|---|
| 10 | USERS | 192.168.0.0/27 | 255.255.255.224 |
| 20 | SERVERS | 192.168.0.32/28 | 255.255.255.240 |
| 30 | IT | 192.168.0.48/28 | 255.255.255.240 |
| 40 | HR | 192.168.0.64/28 | 255.255.255.240 |
| 50 | FINANCE | 192.168.0.80/28 | 255.255.255.240 |
| 60 | SALES | 192.168.0.96/29 | 255.255.255.248 |
| 70 | GUEST | 192.168.0.104/29 | 255.255.255.248 |
| 99 | MANAGEMENT | 192.168.0.112/28 | 255.255.255.240 |

## Firewall Transit VLANs

| VLAN ID | Name | Purpose |
|---:|---|---|
| 901 | FW-TRANSIT-A | Firewall transit on HQ-CORE-01 |
| 902 | FW-TRANSIT-B | Firewall transit on HQ-CORE-02 |

## Core Switch Virtual IPs

The following HSRP virtual IP addresses are configured on HQ-CORE-01:

| VLAN | Virtual IP |
|---:|---|
| 10 | 192.168.0.1 |
| 20 | 192.168.0.33 |
| 30 | 192.168.0.49 |
| 40 | 192.168.0.65 |
| 50 | 192.168.0.81 |
| 60 | 192.168.0.97 |
| 70 | 192.168.0.105 |
| 99 | 192.168.0.113 |

## Notes

- This document records the current documented design.
- Transit VLANs and physical connections must be verified against the running topology.
- Routing and failover functionality will be documented separately after testing.
