# Network Device Configurations

This directory stores sanitized configuration exports collected from the Enterprise Multi-Site Network & Security Infrastructure Lab.

The repository documents a multi-site network connecting Cairo HQ and the Alexandria branch. Configuration files should reflect the actual device configuration collected from GNS3, Cisco IOS, and FortiGate appliances.

**Important:** A planned configuration is not evidence of a deployed configuration. Only publish files collected from the lab, and clearly distinguish collected configuration from independently verified behavior.

## 1. Configuration Inventory

The following files are intended for collection and review. Their listing does not mean that the files already exist or that their configurations have been verified.

| Device / Component | Suggested Filename | Configuration Scope |
|---|---|---|
| Cairo Core Switch 1 | `HQ-CORE-01.txt` | VLANs, SVIs, HSRP, EtherChannel, STP, OSPF, routing, DHCP relay |
| Cairo Core Switch 2 | `HQ-CORE-02.txt` | VLANs, SVIs, HSRP, EtherChannel, STP, OSPF, routing, DHCP relay |
| Alexandria Core Switch | `ALEX-CORE.txt` | VLANs, SVIs, routed links, OSPF, summaries, default route |
| Alexandria ASBR | `ALEX-ASBR.txt` | OSPF, RIP, redistribution, route tagging and filtering |
| Cairo ISP-A Router | `ISP-A.txt` | WAN addressing and routing toward the simulated Internet edge |
| Cairo ISP-B Router | `ISP-B.txt` | WAN addressing and routing toward the simulated Internet edge |
| Internet Edge Router | `INET-EDGE.txt` | Simulated external test networks and routing |
| Cairo FortiGate | `FG-HQ-sanitized.conf` | Interfaces, routes, firewall policies, VIPs, HA, VPN, link monitors |
| Alexandria FortiGate | `FG-ALEX-sanitized.conf` | Interfaces, routes, firewall policies, VPN and relevant security settings |

Use filenames that match the actual device hostnames where possible. If the device hostname differs from the suggested filename, preserve the real hostname and document the difference.

## 2. Required Collection Procedure

Collect the configuration directly from the relevant device before publishing it.

### 2.1 Cisco IOS Devices

For Cisco IOS devices, collect the relevant running configuration and supporting operational output.

Useful read-only commands include:

```text
show running-config
show ip interface brief
show vlan brief
show interfaces trunk
show etherchannel summary
show spanning-tree summary
show standby brief
show ip ospf neighbor
show ip route
show ip protocols
```

Run only commands supported by the device and IOS image. Include only the outputs relevant to the configuration being documented.

A configuration export shows what is configured. Operational commands show the state observed when the commands were executed. Keep these forms of evidence distinct.

### 2.2 FortiGate Devices

Collect the required FortiGate configuration from the actual lab appliances.

Review the following areas where applicable:

- Interface addressing and aliases.
- Static and dynamic routing configuration.
- Firewall policies and address objects/groups.
- Virtual IPs and external service publishing.
- HA settings and heartbeat interfaces.
- IPsec Phase 1 and Phase 2 configuration.
- Link-monitor settings and their targets.
- Relevant security profiles.

Do not publish secrets or private material. Where a command or configuration section includes sensitive values, sanitize it before adding the file to GitHub.

The exact collection method should match the FortiOS version and the configuration format supported by the appliance.

## 3. Configuration Sanitization and Security

**Never publish the following:**

- Administrator passwords or password hashes.
- IPsec pre-shared keys.
- Private keys or private certificates containing key material.
- API tokens, access tokens, or session credentials.
- SNMP community strings or other authentication secrets.
- Sensitive personal data or credentials embedded in comments.
- Other secrets that could enable unauthorized access.

Before committing a configuration file:

1. Inspect the complete file, including comments and embedded scripts.
2. Replace sensitive values with explicit placeholders such as `<REDACTED>`.
3. Preserve the configuration structure where safe, but do not imply that a redacted file can be imported without modification.
4. Check the Git diff to confirm that no secrets were introduced.
5. Verify that the file is appropriate for public release.

A file should be described as **sanitized** only after it has been reviewed for sensitive information.

## 4. Verification Status

Use the following status labels in filenames, documentation, or a configuration inventory when necessary:

- **Collected:** Exported from the actual lab device.
- **Sanitized:** Reviewed and edited to remove sensitive information.
- **Reviewed:** Checked for consistency with the documented design.
- **Tested:** The relevant configuration behavior was tested, with evidence recorded separately.
- **Not Verified:** The configuration or its operational behavior has not been independently confirmed.

These labels are not interchangeable. For example, a configuration can be collected and sanitized without proving that the resulting network behavior is correct.

## 5. Known Configuration Review Priorities

The following items require careful reconciliation with the live lab before configuration changes are made:

- The Cairo core-to-FortiGate transit links using `192.168.0.128/30` and `192.168.0.136/30`.
- The Alexandria routed links and the discrepancy between the planned addresses and the recorded `ALEX-CORE` interface configuration.
- The Alexandria SVIs that were recorded as `down/down`.
- The DMZ addressing conflict involving `192.168.0.145` and `192.168.0.146`.
- The intended OSPF areas, passive-interface behavior, route summaries, and RIP/OSPF redistribution.
- The WAN1 and WAN2 default routes, link-monitor status, and failover/recovery behavior.
- The FortiGate WAN2 VIP issue, where the recorded debug flow showed a reverse-path check failure after destination NAT.

These are review priorities, not instructions to change the configuration immediately. Compare the live configuration, connected peers, routing state, and relevant test results before applying any change.

## 6. Relationship to Verification Evidence

Configuration files should be linked to the relevant verification documentation when that relationship is established.

- [Routing Verification](../Verification/routing.md)
- [Dual-WAN Failover and Recovery Verification](../Verification/dual-wan-verification.md)
- [IPsec VPN Verification](../Verification/ipsec-vpn.md)
- [FortiGate HA Verification](../Verification/fortigate-ha-verification.md)
- [FortiGate Firewall Policies and Connectivity Verification](../Documentation/fortigate-firewall-policies.md)
- [Network Topology](../Documentation/network-topology.md)
- [IP Addressing and VLANs](../Documentation/ip-addressing-vlans.md)

A configuration file supports review of the deployed settings. It does not replace test evidence for routing, failover, application access, or security behavior.

## 7. Current Repository Status

This README defines the expected configuration collection and publication process.

Do not assume that the inventory above represents files already committed to this directory. Update this section after actual configuration exports have been added, reviewed, and sanitized.

When a configuration file is added, record:

- Device hostname.
- Platform and software version, where known.
- Collection date, where known.
- Whether the file has been sanitized.
- Whether the configuration has been reviewed.
- Which behaviors, if any, have been independently tested.

**Publication rule:** Only describe a configuration as collected, reviewed, sanitized, or tested when the corresponding work has actually been completed and supported by evidence.
