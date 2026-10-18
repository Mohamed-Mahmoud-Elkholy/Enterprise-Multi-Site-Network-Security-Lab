# Network Device Configurations

This directory contains configuration files for network devices used in the Enterprise Multi-Site Network Security Lab.

The inventory below documents the configuration files prepared locally. A file is considered part of the GitHub repository only after it has been uploaded and committed.

## 1. Core Switches

| Device | Configuration File |
|---|---|
| HQ-CORE-01 | `HQ-CORE-01-running-config.txt` |
| HQ-CORE-02 | `HQ-CORE-02-running-config.txt` |
| ALEX-CORE | `ALEX-CORE-running-config.txt` |

These switches provide the core network infrastructure for Cairo HQ and the Alexandria branch.

## 2. Access Switches

| Device | Configuration File |
|---|---|
| ACC-1 | `ACC-1-running-config.txt` |
| ACC-2 | `ACC-2-running-config.txt` |
| ALEX-ACC | `ALEX-ACC-running-config.txt` |
| SRV-1 | `SRV-1-running-config.txt` |
| SRV-2 | `SRV-2-running-config.txt` |

SRV-1 and SRV-2 are switches connected to the server infrastructure. They are not server machines.

## 3. Routing Devices

| Device | Configuration File |
|---|---|
| ALEX-ASBR | `ALEX-ASBR-running-config.txt` |
| INET-EDGE | `INET-EDGE-running-config.txt` |
| ISP-A | `ISP-A-running-config.txt` |
| ISP-B | `ISP-B-running-config.txt` |
| LEG-R1 | `LEG-R1-running-config.txt` |
| LEG-R2 | `LEG-R2-running-config.txt` |

These devices represent the routing and WAN components included in the lab.

## 4. Configuration File Status

The local folder screenshot shows 14 configuration files:

- 3 core switches
- 5 access switches, including SRV-1 and SRV-2
- 6 routing devices

The filenames and file sizes are visible in the local folder listing. The configuration contents have not been independently validated as complete or correct.

The following FortiGate files are not included in this inventory because they were previously observed as empty files:

- `FG-ALEX-01-config.txt`
- `FG-HQ-01-config.txt`
- `FG-HQ-02-config.txt`

Do not add these files to the published inventory until they contain valid configuration content.

## 5. Security Considerations

Before publishing configuration files, review and redact sensitive information, including:

- Passwords and password hashes
- VPN pre-shared keys and private keys
- SNMP community strings
- Authentication secrets and tokens
- Certificates or other sensitive security material
- Internal addressing or operational details that should not be public

Use placeholders such as `<REDACTED>` where necessary. Never publish credentials or configuration data without authorization.

## 6. Validation Guidelines

Before committing a configuration file:

1. Confirm that the file is not empty.
2. Verify the device hostname and intended role.
3. Confirm that the configuration belongs to the correct lab device.
4. Remove sensitive information.
5. Check that the documented configuration reflects the current lab state.
6. Update this inventory whenever files are added, removed, or renamed.

## 7. Repository Status

This README documents the local configuration inventory shown during project preparation. It does not confirm that these files have already been uploaded to GitHub.

Only files committed under the `configs/` directory are available to repository visitors.

## 8. Project Scope

These configurations support the Enterprise Multi-Site Network Security Lab, covering the Cairo HQ network, Alexandria branch, core and access switching, routing infrastructure, and WAN connectivity.

Refer to the project topology, IP addressing documentation, and verification reports for additional context about the network design and tested behavior.
