# FortiGate Firewall Policies

## 1. Overview

This document records the firewall policies configured on the FortiGate devices in the **Enterprise Multi-Site Network & Security Infrastructure Lab**.

The lab uses FortiGate firewalls at Cairo HQ and the Alexandria branch to control Internet access, inter-site traffic, and access to the DMZ web server.

The documented configuration and verification results distinguish between:
- **Completed:** Implemented and verified in the lab.
- **Known Issue:** A specific failure remains under investigation.
- **Configuration Record:** A policy or setting has been identified in the available configuration evidence.

The VPN and internal LAN-to-DMZ connectivity are completed parts of the project. The outstanding issue documented here is external HTTPS access through WAN2 to the DMZ web server.

## 2. Firewall Policy Inventory

### 2.1 Cairo HQ — FG-HQ-1

| Policy Name | Purpose | Status |
|---|---|---|
| `LAN_TO_INTERNET_WAN1` | Internal network Internet access through WAN1 | Configured; NAT enabled |
| `LAN_TO_INTERNET_WAN2` | Internal network Internet access through WAN2 | Configured; NAT enabled |
| `WAN1-to-WEB01-HTTPS` | External HTTPS access to the DMZ web server through WAN1 | Configured; WAN1 external TCP connection opened during testing |
| `WAN2-to-WEB01-HTTPS` | External HTTPS access to the DMZ web server through WAN2 | Configured; external access has a documented reverse-path failure |
| `HQ-to-Alexandria` | Traffic between HQ and the Alexandria branch | Configured |
| `HO-TO-ALEX-WAN2` | Inter-site traffic associated with the second WAN path | Configured |
| `IT_SERVERS_TO_WEB01_DMZ` | IT/server network access to `WEB01-DMZ` | Configured; HTTP, HTTPS, and PING-ICMP services identified |

The HQ policy list also showed an implicit deny rule at the bottom. Traffic that does not match an applicable allow policy is subject to the implicit deny behavior.

### 2.2 Alexandria — FG-ALEX-01

| Policy Name | Purpose | Status |
|---|---|---|
| `HQ-WAN1-TO-ALEX-LAN` | Traffic from HQ over the WAN1-associated path toward Alexandria LAN | Configured |
| `HQ-WAN2-TO-ALEX-LAN` | Traffic from HQ over the WAN2-associated path toward Alexandria LAN | Configured |
| `Alexandria-to-HQ` | Traffic from Alexandria toward HQ | Configured |
| `ALEX-TO-HO-WAN2` | Inter-site traffic associated with the second WAN path | Configured |
| `LAN_To_WAN1` | Alexandria LAN Internet access through WAN1 | Configured; NAT enabled |
| `LAN_To_WAN2` | Alexandria LAN Internet access through WAN2 | Configured; NAT enabled |

The policy names are preserved as observed in the lab. Their names describe their intended purpose and are not used as a substitute for actual test results.

## 3. DMZ Access Policy

### `IT_SERVERS_TO_WEB01_DMZ`

This policy controls the documented internal IT/server access path to the DMZ web server.

| Field | Documented Information |
|---|---|
| Policy Name | `IT_SERVERS_TO_WEB01_DMZ` |
| Destination | `WEB01-DMZ` |
| DMZ web server IP | `192.168.0.146` |
| Services | HTTP, HTTPS, PING-ICMP |
| Status | Completed |
| Evidence | Existing lab configuration and DMZ verification screenshot |

**Result:** Internal LAN-to-DMZ connectivity is completed. It must not be classified as an outstanding issue merely because external access through WAN2 is failing.

The internal DMZ connectivity result is separate from the external WAN-to-DMZ publishing policies documented below.

## 4. Internet Access and NAT

The following Internet access policies were identified:

- HQ: `LAN_TO_INTERNET_WAN1`
- HQ: `LAN_TO_INTERNET_WAN2`
- Alexandria: `LAN_To_WAN1`
- Alexandria: `LAN_To_WAN2`

NAT was observed as enabled on these LAN-to-Internet policies.

Dual-WAN behavior is documented separately from DMZ publishing. A failure involving the WAN2 DMZ VIP must not automatically be interpreted as failure of all WAN2 Internet access.

## 5. Inter-Site Traffic and VPN

The policy inventory includes the following policies associated with traffic between Cairo HQ and Alexandria:

**HQ policies**
- `HQ-to-Alexandria`
- `HO-TO-ALEX-WAN2`

**Alexandria policies**
- `Alexandria-to-HQ`
- `ALEX-TO-HO-WAN2`
- `HQ-WAN1-TO-ALEX-LAN`
- `HQ-WAN2-TO-ALEX-LAN`

### VPN status

**Status: Completed.**

The lab's IPsec VPN implementation and its documented verification are treated as completed project work. The VPN is not listed as an outstanding issue in this firewall policy document.

The separate VPN verification document retains the evidence and test results for the implemented tunnels. The WAN2 external HTTPS problem described below is a distinct DMZ publishing issue and does not change the recorded completion status of the VPN.

## 6. External HTTPS Access to the DMZ

The DMZ web server is:

| Field | Value |
|---|---|
| Server name | `WEB01-DMZ` |
| Server IP | `192.168.0.146` |
| Published service | HTTPS |
| External port | TCP/443 |
| Internal destination port | TCP/443 |

### 6.1 Recorded VIP Configuration

| VIP | Interface | External IP | Mapped IP | Port Mapping |
|---|---|---|---|---|
| `VIP-WEB01-HTTPS` | `WAN1-ISP-A (port1)` | `198.51.100.2` | `192.168.0.146` | TCP/443 → 443 |
| `VIP-WEB01-HTTPS-WAN2` | `WAN2-ISP-B (port2)` | `203.0.113.2` | `192.168.0.146` | TCP/443 → 443 |

These addresses reflect the recorded lab configuration. The external VIP addresses match the documented FortiGate WAN interface addresses, so this configuration detail should be checked against the live configuration before any change is made. It is recorded here as observed, not declared correct or changed.

### 6.2 WAN1 — External HTTPS Path

The external TCP test from `INET-EDGE` to `198.51.100.2:443` opened a TCP connection. The WAN1 traffic logs showed traffic matching `WAN1-to-WEB01-HTTPS`, with an accepted session start.

A TCP connection opening is evidence that the TCP attempt reached an accepting endpoint; it does not, by itself, prove successful HTTPS application access. The test result is therefore recorded accurately without claiming that the web page was fully validated.

### 6.3 WAN2 — Known Issue

The external TCP test from `INET-EDGE` (`203.0.113.250`) to `203.0.113.2:443` timed out.

The FortiGate debug flow showed:

- The incoming packet arrived on `port2`.
- Destination NAT translated the destination to `192.168.0.146:443`.
- The packet was dropped because the reverse-path check failed.

The relevant routing evidence showed two default route candidates:

- WAN1 through `198.51.100.1`, distance 10 — preferred route.
- WAN2 through `203.0.113.1`, distance 20 — backup route.

The DMZ network `192.168.0.144/28` was shown as connected through `port7`, which is the documented DMZ-facing interface.

**Current result:** WAN2 external HTTPS publishing remains unresolved. The observed reverse-path failure is the confirmed drop reason in the captured debug flow; the underlying cause has not yet been established.

No arbitrary routing or firewall changes should be made solely from this evidence. Any change must be based on the live routing and policy configuration and followed by a repeat test.

## 7. Verification Results

| Test | Result | Notes |
|---|---|---|
| ICMP from FG-HQ-1 to `192.168.0.146` | Passed | 5 packets received; 0% packet loss |
| Internal LAN-to-DMZ connectivity | Completed | Existing lab evidence and screenshot |
| VPN implementation and documented verification | Completed | See the VPN verification document |
| External TCP attempt through WAN1 | TCP connection opened | HTTPS application response not established by this test alone |
| External TCP attempt through WAN2 | Failed | Connection timed out |
| WAN2 VIP destination NAT | Observed | Translation to `192.168.0.146:443` |
| WAN2 reverse-path check | Failed | Packet dropped according to debug flow |

## 8. Remaining Work

The outstanding issue covered by this document is limited to:

**Resolve and verify external HTTPS access through WAN2 to `WEB01-DMZ` (`192.168.0.146:443`).**

The investigation should establish why the reverse-path check rejects the incoming WAN2 VIP traffic, confirm the correct routing behavior for the external source, and retest the complete connection after any justified correction.

The following work is not classified as incomplete in this document:
- The implemented VPN.
- Internal LAN-to-DMZ connectivity.
- The existence of the documented firewall policies.
- The WAN1 external TCP connection attempt.

The WAN1 TCP test must not be overstated as a successful HTTPS application test, and the WAN2 issue must not be generalized into a claim that the entire DMZ or VPN implementation is incomplete.

## 9. Evidence and Documentation

Relevant evidence is maintained in the project repository, including the existing configuration screenshots and verification documents.

When recording the next WAN2 test, capture:
- Test source and destination.
- Protocol and port.
- Result of the external TCP/HTTPS attempt.
- Matching FortiGate traffic log.
- Debug-flow output.
- Routing-table details relevant to the external source and DMZ destination.

This evidence will allow the specific WAN2 publishing issue to be closed without reopening or misclassifying already completed project components.
