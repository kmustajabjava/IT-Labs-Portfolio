# Extended Numbered and Named ACL

## Objective

Configure, apply, and verify both **extended numbered** and **extended named IPv4 ACLs** to control access to specific network services.

## Scenario

Two employees require different services from a server:

- **PC1** requires FTP access.
- **PC2** requires HTTP/web access.
- Both PCs must be able to ping the server.
- PC1 and PC2 should not be able to communicate directly with each other.

---

# Part 1 — Extended Numbered ACL

## 1. Create ACL 100

The first extended ACL number was selected from the range `100–199`.

PC1's network:

```text
172.22.34.64/27
```

Wildcard mask:

```text
0.0.0.31
```

Server:

```text
172.22.34.62
```

### Permit FTP

```text
R1(config)# access-list 100 permit tcp 172.22.34.64 0.0.0.31 host 172.22.34.62 eq ftp
```

### Permit ICMP

```text
R1(config)# access-list 100 permit icmp 172.22.34.64 0.0.0.31 host 172.22.34.62
```

Because extended ACLs have an implicit deny at the end, traffic not matching these statements is denied.

### Verify

```text
R1# show access-lists
```

Expected:

```text
Extended IP access list 100
    10 permit tcp 172.22.34.64 0.0.0.31 host 172.22.34.62 eq ftp
    20 permit icmp 172.22.34.64 0.0.0.31 host 172.22.34.62
```

## 2. Apply ACL 100

The ACL was applied inbound on the interface receiving traffic from PC1's network:

```text
R1(config)# interface gigabitEthernet 0/0
R1(config-if)# ip access-group 100 in
```

## 3. Verify

PC1 was tested for:

- Ping to the Server → **Allowed**
- FTP to the Server → **Allowed**
- Ping to PC2 → **Blocked**

FTP test:

```text
PC> ftp 172.22.34.62
```

---

# Part 2 — Extended Named ACL

A named extended ACL was configured for PC2.

PC2 network:

```text
172.22.34.96/28
```

Wildcard mask:

```text
0.0.0.15
```

## 1. Create HTTP_ONLY ACL

```text
R1(config)# ip access-list extended HTTP_ONLY
```

### Permit HTTP

```text
R1(config-ext-nacl)# permit tcp 172.22.34.96 0.0.0.15 host 172.22.34.62 eq www
```

### Permit ICMP

```text
R1(config-ext-nacl)# permit icmp 172.22.34.96 0.0.0.15 host 172.22.34.62
```

### Verify

```text
R1# show access-lists
```

Expected:

```text
Extended IP access list HTTP_ONLY
    10 permit tcp 172.22.34.96 0.0.0.15 host 172.22.34.62 eq www
    20 permit icmp 172.22.34.96 0.0.0.15 host 172.22.34.62
```

## 2. Apply the named ACL

The ACL was applied inbound on the interface connected to PC2's network:

```text
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# ip access-group HTTP_ONLY in
```

## 3. Verification

From PC2:

- Ping Server → **Allowed**
- HTTP/web access to Server → **Allowed**
- FTP to Server → **Blocked**

## Security Concept

Extended ACLs provide more granular control than standard ACLs because they can filter based on:

- Source IP
- Destination IP
- Protocol
- TCP/UDP port
- ICMP traffic

This makes them suitable for **service-level access control**.

## Key Takeaways

- Standard ACLs mainly filter by source address.
- Extended ACLs can control specific services and protocols.
- Named ACLs make configurations easier to identify and manage.
- Correct ACL placement reduces unnecessary traffic and improves filtering effectiveness.
- Implicit deny should always be considered when designing ACLs.