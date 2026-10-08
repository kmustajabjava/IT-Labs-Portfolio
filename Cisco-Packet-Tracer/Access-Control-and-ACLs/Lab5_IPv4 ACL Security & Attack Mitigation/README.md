# IPv4 ACL Security and Attack Mitigation

## Objective

Configure, apply, and verify multiple IPv4 ACLs to secure router management access, filter network services, control ICMP traffic, and prevent traffic using private RFC 1918 source addresses.

## Scenario

This lab demonstrates several practical uses of IPv4 access control lists in a routed network.

The security requirements include:

- Restricting SSH management access to authorized administrators
- Protecting services running on PC-A
- Filtering unwanted ICMP traffic
- Restricting traffic entering R3 from its LAN
- Blocking packets using private RFC 1918 addresses as external source addresses
- Allowing legitimate SSH traffic from the `10.0.0.0/8` network to return to PC-C

Before configuring the ACLs, end-to-end connectivity was verified.

---

# Part 1 — Secure SSH Remote Management

## Security Requirement

The remote-management policy was:

- **PC-A (`192.168.1.3`)** cannot establish SSH sessions to any router.
- **PC-C (`192.168.3.3`)** can establish SSH sessions to all routers.

This protects router management interfaces from unauthorized hosts while maintaining access for the designated administration workstation.

## 1. Configure ACL 101

On the routers, ACL 101 was configured to identify SSH traffic based on the source host.

```text
access-list 101 permit tcp host 192.168.3.3 any eq 22
access-list 101 deny tcp host 192.168.1.3 any eq 22
access-list 101 permit ip any any
```

### ACL logic

| Rule | Purpose |
|---|---|
| Permit PC-C → TCP/22 | Allows authorized SSH management |
| Deny PC-A → TCP/22 | Blocks unauthorized SSH management |
| Permit all other IP traffic | Prevents unrelated traffic from being blocked |

## 2. Apply the ACL to VTY Lines

The ACL was applied to the router VTY lines:

```text
line vty 0 4
access-class 101 in
```

The `access-class` command restricts which source addresses can access the router through the VTY lines.

## Verification

SSH connectivity was tested from the appropriate hosts.

Expected behavior:

- PC-C → Router SSH: **Allowed**
- PC-A → Router SSH: **Blocked**

---

# Part 2 — Filter Incoming Services on R1

## Security Requirements

The R1 security policy requires:

- Outside hosts can access **DNS** on PC-A.
- Outside hosts can access **SMTP** on PC-A.
- Outside hosts can access **FTP** on PC-A.
- Outside hosts cannot access **HTTPS** on PC-A.
- PC-C can SSH to R1's `S0/0/0` interface.

PC-A:

```text
192.168.1.3
```

PC-C:

```text
192.168.3.3
```

R1 `S0/0/0`:

```text
10.1.1.1
```

## 1. Configure ACL 110

```text
access-list 110 permit udp any host 192.168.1.3 eq 53
access-list 110 permit tcp any host 192.168.1.3 eq 53
access-list 110 permit tcp any host 192.168.1.3 eq 25
access-list 110 permit tcp any host 192.168.1.3 eq 21
access-list 110 deny tcp any host 192.168.1.3 eq 443
access-list 110 permit tcp host 192.168.3.3 host 10.1.1.1 eq 22
```

### Service mapping

| Port | Service | Action |
|---|---|---|
| 53/UDP | DNS | Permit |
| 53/TCP | DNS | Permit |
| 25/TCP | SMTP | Permit |
| 21/TCP | FTP | Permit |
| 443/TCP | HTTPS | Deny |
| 22/TCP | SSH from PC-C | Permit |

## 2. Apply ACL 110

The ACL was applied inbound on R1's serial interface receiving outside traffic:

```text
interface serial 0/0/0
ip access-group 110 in
```

---

# Part 3 — ICMP Filtering

The ACL was then extended to control incoming ICMP traffic.

The requirements were:

- Permit ICMP echo replies.
- Permit destination-unreachable messages.
- Deny other incoming ICMP packets.
- Permit all other IP traffic.

The following statements were added:

```text
access-list 110 permit icmp any any echo-reply
access-list 110 permit icmp any any unreachable
access-list 110 deny icmp any any
access-list 110 permit ip any any
```

## Why allow echo replies instead of echo requests?

The router needs to receive **responses** to legitimate ping requests initiated from inside the network.

Allowing arbitrary incoming echo requests would allow external hosts to initiate ping traffic toward internal systems, which was not required by the security policy.

---

# Part 4 — Restrict Traffic Entering R3 from the LAN

A standard ACL was configured on R3 to allow only addresses belonging to its internal LAN.

R3 LAN:

```text
192.168.3.0/24
```

## 1. Configure ACL 1

```text
access-list 1 permit 192.168.3.0 0.0.0.255
```

Because a standard ACL has an implicit deny, sources outside this network are denied.

## 2. Apply the ACL

The ACL was applied inbound on R3's LAN interface:

```text
interface gigabitEthernet 0/1
ip access-group 1 in
```

This prevents unauthorized source addresses from entering R3 through the internal LAN interface.

---

# Part 5 — RFC 1918 Source Address Filtering

The final section focused on filtering packets using private IPv4 source addresses.

RFC 1918 private ranges are:

- `10.0.0.0/8`
- `172.16.0.0/12`
- `192.168.0.0/16`

The policy required:

- Allow SSH traffic from `10.0.0.0/8` to return to PC-C.
- Deny other traffic sourced from `10.0.0.0/8`.
- Deny traffic sourced from `172.16.0.0/12`.
- Deny traffic sourced from `192.168.0.0/16`.
- Permit all other traffic.

## 1. Configure ACL 100

```text
access-list 100 permit tcp 10.0.0.0 0.255.255.255 eq 22 host 192.168.3.3
access-list 100 deny ip 10.0.0.0 0.255.255.255 any
access-list 100 deny ip 172.16.0.0 0.15.255.255 any
access-list 100 deny ip 192.168.0.0 0.0.255.255 any
access-list 100 permit ip any any
```

### Important ACL ordering

The SSH permit statement must appear **before** the `10.0.0.0/8` deny statement.

Otherwise, legitimate SSH traffic from the `10.0.0.0/8` range would also be denied.

## 2. Apply ACL 100

The ACL was applied inbound on R3's serial interface:

```text
interface serial 0/0/1
ip access-group 100 in
```

## Verification

The ACL configuration and interface placement can be checked using:

```text
show access-lists
show running-config
show ip interface serial 0/0/1
```

The traffic entering `Serial0/0/1` was then tested against the ACL rules.

---

# Security Concepts Demonstrated

This lab demonstrates several practical network-security controls:

### 1. Management-plane protection

SSH access to routers was restricted to an authorized administration host.

### 2. Service-based filtering

DNS, SMTP, FTP, HTTPS, and SSH were selectively permitted or denied using TCP/UDP ports.

### 3. ICMP filtering

Unwanted ICMP traffic was blocked while necessary error messages and echo replies remained permitted.

### 4. Source-address filtering

Traffic entering the network was restricted according to expected source networks.

### 5. RFC 1918 anti-spoofing

Private source addresses were blocked at the network boundary to reduce the risk of spoofed internal addresses entering from an external network.

## Key Takeaways

- ACLs can protect both the **management plane** and **data plane** of a network.
- Extended ACLs provide granular control over protocols, addresses, and ports.
- ACL statement order is critical.
- Private RFC 1918 addresses should generally not appear as source addresses when traffic enters from an external network.
- Verification before and after applying an ACL helps prevent accidental connectivity loss.