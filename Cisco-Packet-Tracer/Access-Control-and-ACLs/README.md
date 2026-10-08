# Access Control and ACLs

A collection of Cisco Packet Tracer labs focused on **IPv4 and IPv6 Access Control Lists (ACLs)** for network access control, service filtering, router management protection, and basic attack mitigation.

These labs demonstrate how ACLs can be used to control traffic based on **source/destination addresses, protocols, ports, and services**.

## Labs Included

| Lab | Topic | Main Security Focus |
|---|---|---|
| **01** | Named Standard ACL | Restrict access to a specific server |
| **02** | Standard ACL Planning & Implementation | Source-based traffic filtering |
| **03** | Extended Numbered & Named ACL | Protocol and service-specific access control |
| **04** | Extended ACL Service Filtering | HTTP/HTTPS, FTP, and ICMP filtering |
| **05** | IPv4 ACL Security & Attack Mitigation | SSH protection, ICMP filtering, RFC 1918 filtering |
| **06** | IPv6 ACL HTTP/HTTPS & ICMP Filtering | IPv6 service filtering and DoS/DDoS mitigation |

## Skills Demonstrated

- Configuring standard IPv4 ACLs
- Configuring extended IPv4 ACLs
- Configuring named and numbered ACLs
- Configuring IPv6 ACLs
- Filtering traffic by source and destination IP
- Filtering TCP and UDP services using port numbers
- Filtering ICMP traffic
- Protecting router VTY/SSH management access
- Applying ACLs inbound and outbound
- Selecting appropriate ACL placement
- Verifying ACL configuration and traffic behavior
- Using ACL counters for troubleshooting
- Blocking unwanted or potentially malicious traffic
- Filtering private RFC 1918 source addresses

## Security Concepts

### Management Access Control

ACLs can restrict remote router management so that only authorized hosts can establish SSH sessions.

### Service-Based Filtering

Extended ACLs can control individual services such as:

- HTTP
- HTTPS
- FTP
- DNS
- SMTP
- SSH

### ICMP Filtering

ICMP ACLs can be used to control ping and other ICMP traffic while allowing required network traffic to continue.

### RFC 1918 Source Filtering

Private IPv4 source ranges can be filtered at network boundaries to reduce the risk of spoofed internal addresses entering from an external network.

### IPv6 Security

IPv6 ACLs can provide similar traffic-control capabilities for IPv6 networks, including filtering application services and ICMP traffic.

### ACL Placement

The labs demonstrate that ACL placement depends on the security requirement. Filtering may be performed close to the **source** to stop unwanted traffic early or close to the **destination** when protecting a particular network resource.

## Verification Commands

Common verification commands used throughout the labs include:

```text
show access-lists
show ip access-lists
show running-config
show ip interface
show ipv6 access-list
show ipv6 interface
show ipv6 interface brief
```

ACL counters were also used to confirm whether traffic matched specific permit or deny statements.

## Tools

- **Cisco Packet Tracer**
- Cisco IOS CLI
- IPv4
- IPv6
- TCP/UDP
- ICMP
- SSH

## Overall Learning Outcomes

Through these labs, I practiced implementing network access-control policies using Cisco ACLs and learned how to:

1. Translate security requirements into ACL rules.
2. Select standard or extended ACLs according to the requirement.
3. Use named and numbered ACLs.
4. Control access to network services using protocol and port information.
5. Protect router management interfaces.
6. Apply ACLs to the appropriate interface and direction.
7. Verify and troubleshoot ACL behavior.
8. Use ACLs as a basic network-security control against unauthorized and potentially malicious traffic.

> **Note:** Credentials used in the original Packet Tracer activities are intentionally not included in this repository.