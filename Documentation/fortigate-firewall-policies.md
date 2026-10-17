# FortiGate Firewall Policies

## 1. Overview

This document records the firewall policies configured on the FortiGate devices in the **Enterprise Multi-Site Network & Security Infrastructure Lab**.

The lab uses FortiGate firewalls at Cairo HQ and the Alexandria branch to control Internet access, inter-site traffic, and access to the DMZ web server.

The documentation is based on the policy names and configuration details observed in the lab screenshots. Any configuration detail that has not been confirmed is explicitly marked as requiring verification.

## 2. Firewall Policy Inventory

### 2.1 Cairo HQ — FG-HQ-1

The following policies were identified in the HQ firewall policy list.

| Policy Name | Purpose | Documented Status |
|---|---|---|
| `LAN_TO_INTERNET_WAN1` | Internal network Internet access through WAN1 | Policy identified; NAT enabled |
| `LAN_TO_INTERNET_WAN2` | Internal network Internet access through WAN2 | Policy identified; NAT enabled |
| `WAN1-to-WEB01-HTTPS` | HTTPS access associated with the DMZ web server | Policy identified; outgoing interface requires review |
| `WAN2-to-WEB01-HTTPS` | HTTPS access associated with the DMZ web server | Policy identified; full settings require verification |
| `HQ-to-Alexandria` | Traffic between HQ and the Alexandria branch | Policy identified; full settings require verification |
| `HO-TO-ALEX-WAN2` | Inter-site traffic associated with the second WAN path | Policy identified; exact interfaces and match conditions require verification |
| `IT_SERVERS_TO_WEB01_DMZ` | IT/server network access to `WEB01-DMZ` | Policy identified; services include HTTP, HTTPS, and PING-ICMP |

The HQ policy list also showed an implicit deny rule at the bottom. This is the default policy behavior when traffic does not match an applicable policy.

### 2.2 Alexandria — FG-ALEX-01

The following policies were identified in the Alexandria firewall policy list.

| Policy Name | Purpose | Documented Status |
|---|---|---|
| `HQ-WAN1-TO-ALEX-LAN` | Traffic from HQ over the WAN1-associated path toward Alexandria LAN | Policy identified; full settings require verification |
| `HQ-WAN2-TO-ALEX-LAN` | Traffic from HQ over the WAN2-associated path toward Alexandria LAN | Policy identified; full settings require verification |
| `Alexandria-to-HQ` | Traffic from Alexandria toward HQ | Policy identified; full settings require verification |
| `ALEX-TO-HO-WAN2` | Inter-site traffic associated with the second WAN path | Policy identified; exact interfaces and match conditions require verification |
| `LAN_To_WAN1` | Alexandria LAN Internet access through WAN1 | Policy identified; NAT enabled |
| `LAN_To_WAN2` | Alexandria LAN Internet access through WAN2 | Policy identified; NAT enabled |

**Important:** The policy names indicate the intended traffic paths, but names alone do not prove that routing, VPN selection, or failover works correctly.

## 3. DMZ Access Policy

### `IT_SERVERS_TO_WEB01_DMZ`

This policy was identified on the HQ FortiGate.

| Field | Documented Information |
|---|---|
| Policy Name | `IT_SERVERS_TO_WEB01_DMZ` |
| Destination | `WEB01-DMZ` |
| Services | HTTP, HTTPS, PING-ICMP |
| Action | Requires confirmation from the complete policy settings |
| Incoming Interface | Requires confirmation |
| Outgoing Interface | Requires confirmation |
| NAT | Requires confirmation |
| Security Profiles | Not documented as enabled |

The policy provides a documented control point for IT/server access to the DMZ web server. Its actual permitted source objects, interface direction, action, and NAT settings must be taken from the running configuration before making further claims.

## 4. Internet Access and NAT

The following Internet access policies were identified:

- HQ: `LAN_TO_INTERNET_WAN1`
- HQ: `LAN_TO_INTERNET_WAN2`
- Alexandria: `LAN_To_WAN1`
- Alexandria: `LAN_To_WAN2`

NAT was observed as enabled on these LAN-to-Internet policies.

The existence of these policies does not, by itself, prove successful Internet access or automatic WAN failover. Those behaviors require separate connectivity tests and route verification.

## 5. Inter-Site Traffic and VPN

The policy inventory includes policies associated with traffic between Cairo HQ and Alexandria.

**HQ policies**
- `HQ-to-Alexandria`
- `HO-TO-ALEX-WAN2`

**Alexandria policies**
- `Alexandria-to-HQ`
- `ALEX-TO-HO-WAN2`
- `HQ-WAN1-TO-ALEX-LAN`
- `HQ-WAN2-TO-ALEX-LAN`

The lab also contains WAN1-associated and WAN2-associated IPsec VPN tunnels. Tunnel status and firewall policy behavior must be verified separately: a tunnel being up does not prove that the required application traffic can pass through it.

## 6. Configuration Review Items

The following items require review before the policy configuration can be considered fully documented:

1. **`WAN1-to-WEB01-HTTPS`:** The captured policy view appeared to show `WAN2-ISP-B` as the outgoing interface. Compare this with the intended design and correct it only if the running configuration confirms a mismatch.
2. Confirm the incoming and outgoing interfaces for each inter-site policy.
3. Record the exact source and destination address objects or groups for each policy.
4. Confirm the action, service list, NAT setting, and any attached security profiles from the complete policy settings.
5. Check for overlapping or duplicate policies and confirm the intended policy order.
6. Review the configured policy names for consistency. Existing names are preserved here as displayed; naming changes should be made deliberately in the lab.

## 7. Verification Plan

| Test | Required Evidence |
|---|---|
| HQ Internet access through WAN1 | Client connectivity test, matching policy log, and routing evidence |
| HQ Internet access through WAN2 | Client connectivity test, matching policy log, and routing evidence |
| Alexandria Internet access | Client connectivity test and matching policy log |
| HQ-to-Alexandria connectivity | Successful test between the intended source and destination networks |
| VPN path behavior | Tunnel status, route selection, and application connectivity |
| DMZ HTTP/HTTPS access | Connection tests to the intended web server ports and corresponding firewall logs |
| Denied traffic | A test of a specifically prohibited flow and evidence that it was denied |

For each test, record the date, source, destination, protocol/port, result, and relevant evidence filename.

## DMZ HTTPS Verification Results

### Internal Connectivity

The following tests were performed from `FG-HQ-1`:

| Test | Result |
|---|---|
| ICMP to `192.168.0.146` | Passed — 5 packets received, 0% packet loss |
| TCP connection to `192.168.0.146:443` | Connection established, then closed by the remote host |
| WAN1 VIP configuration | DNAT to `192.168.0.146:443` observed in the configuration and debug output |
| WAN1 external TCP test | TCP connection opened from `INET-EDGE` |
| WAN2 external TCP test | Timed out |
| WAN2 debug flow | DNAT was identified, but the packet was dropped with `reverse path check fail` |
| HTTPS application validation | Not completed |

### WAN2 Known Issue

An external TCP connection attempt from `INET-EDGE` (`203.0.113.250`) to the WAN2 VIP (`203.0.113.2:443`) reached `FG-HQ-1` through `port2`.

The debug flow showed destination NAT to `192.168.0.146:443`, followed by a reverse-path check failure and packet drop.

The root cause has not yet been confirmed. Further routing and reverse-path investigation is required before changing the configuration.

### Verification Limitations

Successful ICMP reachability and TCP connection establishment do not prove that the web application is functioning correctly. External HTTPS access through both VIPs remains incompletely verified.

## 8. Evidence Status and Scope

This document records the policy names and selected settings visible in the available lab screenshots. It does not claim that every policy has been independently validated or that every listed traffic flow has passed testing.

The following must remain separate claims until supporting evidence is captured:

- Successful end-to-end application connectivity
- Correct enforcement of every allow/deny policy
- Successful Internet access through both WAN links
- Automatic WAN failover and recovery
- Successful traffic forwarding across both site-to-site VPN paths

Security profiles are not documented as enabled because their attachment to the listed policies has not been established by the evidence used for this document.
