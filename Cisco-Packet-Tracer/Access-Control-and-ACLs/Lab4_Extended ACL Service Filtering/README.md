# Extended ACL — Service-Based Traffic Filtering

## Objective

Configure and apply a **named extended IPv4 ACL** to control access to specific services on remote servers.

## Scenario

Three PCs have different access requirements to two remote servers.

Security policy:

- **PC1** → HTTP and HTTPS access to Server1 and Server2 must be blocked.
- **PC2** → FTP access to Server1 and Server2 must be blocked.
- **PC3** → ICMP/ping access to Server1 and Server2 must be blocked.
- All other IP traffic should remain permitted.

## Network Information

| Device | Address |
|---|---|
| PC1 | `172.31.1.101` |
| PC2 | `172.31.1.102` |
| PC3 | `172.31.1.103` |
| Server1 | `64.101.255.254` |
| Server2 | `64.103.255.254` |

---

# Part 1 — Configure the Extended ACL

The named ACL was created on **RT1**:

```text
RT1(config)# ip access-list extended ACL
```

## 1. Block PC1 HTTP/HTTPS access

### Server1 — HTTP

```text
RT1(config-ext-nacl)# deny tcp host 172.31.1.101 host 64.101.255.254 eq 80
```

### Server1 — HTTPS

```text
RT1(config-ext-nacl)# deny tcp host 172.31.1.101 host 64.101.255.254 eq 443
```

### Server2 — HTTP

```text
RT1(config-ext-nacl)# deny tcp host 172.31.1.101 host 64.103.255.254 eq 80
```

### Server2 — HTTPS

```text
RT1(config-ext-nacl)# deny tcp host 172.31.1.101 host 64.103.255.254 eq 443
```

## 2. Block PC2 FTP access

### Server1

```text
RT1(config-ext-nacl)# deny tcp host 172.31.1.102 host 64.101.255.254 eq 21
```

### Server2

```text
RT1(config-ext-nacl)# deny tcp host 172.31.1.102 host 64.103.255.254 eq 21
```

## 3. Block PC3 ICMP access

### Server1

```text
RT1(config-ext-nacl)# deny icmp host 172.31.1.103 host 64.101.255.254
```

### Server2

```text
RT1(config-ext-nacl)# deny icmp host 172.31.1.103 host 64.103.255.254
```

## 4. Permit all other IP traffic

```text
RT1(config-ext-nacl)# permit ip any any
```

This statement is important because the ACL should block only the specified services while allowing other traffic.

---

# Verify the ACL

```text
RT1# show access-lists
```

Expected structure:

```text
Extended IP access list ACL
    10 deny tcp host 172.31.1.101 host 64.101.255.254 eq www
    20 deny tcp host 172.31.1.101 host 64.101.255.254 eq 443
    30 deny tcp host 172.31.1.101 host 64.103.255.254 eq www
    40 deny tcp host 172.31.1.101 host 64.103.255.254 eq 443
    50 deny tcp host 172.31.1.102 host 64.101.255.254 eq ftp
    60 deny tcp host 172.31.1.102 host 64.103.255.254 eq ftp
    70 deny icmp host 172.31.1.103 host 64.101.255.254
    80 deny icmp host 172.31.1.103 host 64.103.255.254
    90 permit ip any any
```

The configuration can also be inspected with:

```text
RT1# show running-config | begin access-list
```

---

# Part 2 — Apply and Verify the ACL

Because the traffic originates from the `172.31.1.96/27` network and is travelling toward remote networks, the extended ACL should be placed **close to the source**.

The ACL was applied to the appropriate RT1 interface in the **inbound** direction.

```text
RT1(config)# interface <source-facing-interface>
RT1(config-if)# ip access-group ACL in
```

> The exact interface depends on the topology/addressing table of the Packet Tracer activity.

## Verification Tests

### PC1

Test HTTP and HTTPS access to both servers.

Expected:

- PC1 → Server1 HTTP: **Blocked**
- PC1 → Server1 HTTPS: **Blocked**
- PC1 → Server2 HTTP: **Blocked**
- PC1 → Server2 HTTPS: **Blocked**

Other traffic from PC1 should remain permitted.

### PC2

Test FTP access to both servers.

Expected:

- PC2 → Server1 FTP: **Blocked**
- PC2 → Server2 FTP: **Blocked**

### PC3

Test ICMP connectivity to both servers.

Expected:

- PC3 → Server1 ping: **Blocked**
- PC3 → Server2 ping: **Blocked**

## ACL Counters

After testing, use:

```text
RT1# show ip access-lists
```

The match counters show which ACL entries processed the test traffic.

Counters can be cleared with:

```text
RT1# clear access-list counters
```

## Security Concept

This lab demonstrates **service-based network access control** using extended ACLs.

Instead of blocking an entire host or network, the ACL restricts specific combinations of:

**Source → Destination → Protocol → Port**

For example:

```text
PC1 → Server1 → TCP → 80
```

is blocked, while other traffic can remain permitted.

## Key Takeaways

- Extended ACLs provide granular traffic filtering.
- TCP ports can be used to control application services such as HTTP, HTTPS, and FTP.
- ICMP can be filtered independently from TCP/UDP services.
- ACL statement order matters because packets are evaluated sequentially.
- `permit ip any any` allows traffic that does not match the specified restrictions.
- ACL counters are useful for verifying that security policies are actually being triggered.