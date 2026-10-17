# FortiGate Firewall Policies

## 1. Overview

This document describes the firewall policy design for the Enterprise Multi-Site Network & Security Infrastructure Lab.

The lab uses FortiGate firewalls to control traffic between internal VLANs, the Internet, the DMZ, and the Alexandria branch.

## 2. Security Objectives

- Control traffic between network segments.
- Apply explicit source and destination restrictions.
- Permit only the services required by each communication flow.
- Control Internet access through firewall policies and NAT where configured.
- Protect the DMZ by limiting access to published services.
- Support secure connectivity between Cairo HQ and Alexandria through IPsec VPN tunnels.

## 3. Policy Documentation

Each policy should be documented using the following fields:

| Field | Description |
|---|---|
| Policy Name | Configured firewall policy name |
| Incoming Interface | Interface or zone receiving the traffic |
| Outgoing Interface | Interface or zone forwarding the traffic |
| Source | Permitted source addresses or address groups |
| Destination | Destination addresses or address groups |
| Service | Permitted protocols and ports |
| Action | Accept or Deny |
| NAT | Whether source NAT is enabled |
| Security Profiles | Profiles actually attached to the policy |

## 4. Verification

Firewall policies must be validated against the running configuration and traffic tests.

Verification should include:

- Confirming the configured source and destination objects.
- Checking the incoming and outgoing interfaces.
- Confirming the permitted services and policy actions.
- Testing allowed traffic flows.
- Testing denied traffic flows where an explicit restriction is configured.
- Reviewing traffic logs to confirm which policy handled the session.

## 5. Evidence Status

This document defines the policy documentation structure and verification criteria. It does not claim that individual policies or security tests have been verified.

Policy-specific details and test results should be added from the actual FortiGate configuration and captured test evidence.

