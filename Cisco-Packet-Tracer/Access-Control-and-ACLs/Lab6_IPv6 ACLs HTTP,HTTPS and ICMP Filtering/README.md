# IPv6 ACLs — HTTP/HTTPS and ICMP Filtering

## Objective

Configure, apply, and verify IPv6 ACLs to mitigate service-based attacks and ICMP-based denial-of-service traffic.

The lab demonstrates two different ACL placement strategies:

1. Blocking HTTP/HTTPS traffic close to the **source** of the unwanted traffic.
2. Blocking ICMP traffic close to the **destination** because the traffic can originate from many different sources.

---

# Part 1 — Block HTTP and HTTPS Traffic

## Scenario

Network logs indicated that a computer on:

```text
2001:DB8:1:11::/64
```

was repeatedly refreshing a web page and causing a potential **Denial-of-Service (DoS)** attack against Server3.

Server3:

```text
2001:DB8:1:30::30
```

Until the affected client could be identified and cleaned, HTTP and HTTPS access to Server3 needed to be blocked.

## 1. Identify the Source Interface

On R1, the IPv6 interfaces were checked:

```text
R1# show ipv6 interface brief
```

The relevant interface was:

```text
GigabitEthernet0/1
2001:DB8:1:11::1
```

Therefore, `GigabitEthernet0/1` was the interface connected to the source network.

## 2. Create the IPv6 ACL

The named IPv6 ACL `BLOCK_HTTP` was created on R1:

```text
R1(config)# ipv6 access-list BLOCK_HTTP
```

### Block HTTP

```text
R1(config-ipv6-acl)# deny tcp any host 2001:db8:1:30::30 eq 80
```

### Block HTTPS

```text
R1(config-ipv6-acl)# deny tcp any host 2001:db8:1:30::30 eq 443
```

### Permit all other IPv6 traffic

```text
R1(config-ipv6-acl)# permit ipv6 any any
```

The complete ACL was:

```text
ipv6 access-list BLOCK_HTTP
 deny tcp any host 2001:db8:1:30::30 eq 80
 deny tcp any host 2001:db8:1:30::30 eq 443
 permit ipv6 any any
```

### Why `permit ipv6 any any`?

IPv6 ACLs have an implicit deny at the end. Without the final permit statement, other IPv6 traffic could also be blocked.

## 3. Apply the ACL

Because the unwanted traffic originated from the `2001:DB8:1:11::/64` network, the ACL was applied **inbound on the interface closest to the source**:

```text
R1(config)# interface gigabitEthernet 0/1
R1(config-if)# ipv6 traffic-filter BLOCK_HTTP in
```

## 4. Verify the Configuration

The ACL was checked using:

```text
R1# show ipv6 access-list BLOCK_HTTP
```

The interface configuration was checked using:

```text
R1# show ipv6 interface gigabitEthernet 0/1
```

The interface should show `BLOCK_HTTP` as the inbound IPv6 traffic filter.

## 5. Test the ACL

### PC1 → Server3

PC1 was tested using the web browser:

```text
http://[2001:db8:1:30::30]
```

and HTTPS:

```text
https://[2001:db8:1:30::30]
```

Expected:

**Allowed**

### PC2 → Server3

PC2 was tested against the same HTTP/HTTPS services.

Expected:

**Blocked**

### PC2 → Server3 Ping

```text
ping 2001:db8:1:30::30
```

Expected:

**Successful**

This confirms that the ACL blocked only the specified TCP services and did not block general IPv6 traffic.

---

# Part 2 — Block ICMP Traffic

## Scenario

The logs subsequently indicated that Server3 was receiving ping requests from many different IPv6 addresses as part of a potential **Distributed Denial-of-Service (DDoS)** attack.

The requirement was to block ICMP traffic to Server3 while keeping other IPv6 services available.

Server3:

```text
2001:DB8:1:30::30
```

## 1. Create the BLOCK_ICMP ACL

The ACL was configured on R3:

```text
R3(config)# ipv6 access-list BLOCK_ICMP
```

### Block ICMP

```text
R3(config-ipv6-acl)# deny icmp any any
```

### Permit all other IPv6 traffic

```text
R3(config-ipv6-acl)# permit ipv6 any any
```

Complete ACL:

```text
ipv6 access-list BLOCK_ICMP
 deny icmp any any
 permit ipv6 any any
```

## 2. Identify the Destination Interface

R3's IPv6 interfaces were checked:

```text
R3# show ipv6 interface brief
```

The interface connected to Server3's network was:

```text
GigabitEthernet0/0
2001:DB8:1:30::1
```

Server3 was also on the:

```text
2001:DB8:1:30::/64
```

network.

Therefore, `GigabitEthernet0/0` was the interface closest to the destination.

## 3. Apply the ACL

Because ICMP traffic could originate from many different sources, the ACL was placed close to the destination:

```text
R3(config)# interface gigabitEthernet 0/0
R3(config-if)# ipv6 traffic-filter BLOCK_ICMP in
```

## 4. Verify

ACL configuration:

```text
R3# show ipv6 access-list BLOCK_ICMP
```

Interface configuration:

```text
R3# show ipv6 interface gigabitEthernet 0/0
```

The interface should show `BLOCK_ICMP` as the inbound IPv6 traffic filter.

## 5. Test ICMP Filtering

### PC2 → Server3

```text
ping 2001:db8:1:30::30
```

Expected:

**Failed**

### PC1 → Server3

```text
ping 2001:db8:1:30::30
```

Expected:

**Failed**

Both tests confirmed that ICMP traffic was being filtered.

## 6. Verify Other IPv6 Services

From PC1, the Server3 website was accessed using:

```text
http://[2001:db8:1:30::30]
```

and/or:

```text
https://[2001:db8:1:30::30]
```

Expected:

**Website accessible**

This confirmed that the `BLOCK_ICMP` ACL was blocking ICMP while allowing other IPv6 traffic.

---

# ACL Placement Strategy

An important networking concept demonstrated by this lab is that ACL placement depends on the traffic being filtered.

### BLOCK_HTTP

The HTTP/HTTPS ACL was placed close to the **source**:

```text
2001:DB8:1:11::/64
        ↓
    R1 G0/1
        ↓
     Server3
```

This allows the unwanted traffic to be filtered as close as possible to its source.

### BLOCK_ICMP

The ICMP ACL was placed close to the **destination**:

```text
Multiple IPv6 sources
        ↓
       R3
        ↓
     G0/0
        ↓
     Server3
```

Because the ICMP attack could originate from many different addresses, filtering near Server3 ensures that the traffic is blocked regardless of its source.

# Security Concepts Demonstrated

- IPv6 access control lists
- Service-based traffic filtering
- HTTP/HTTPS filtering
- ICMP filtering
- DoS mitigation
- DDoS traffic filtering
- IPv6 interface ACL application
- Source-side ACL placement
- Destination-side ACL placement
- Verification using router CLI commands

## Key Takeaways

- IPv6 ACLs can filter traffic using protocol, source, destination, and service information.
- `permit ipv6 any any` is required when other IPv6 traffic should remain allowed.
- ACL placement should be selected according to the traffic being filtered.
- Service-specific filtering can mitigate application-layer attacks such as repeated HTTP requests.
- ICMP filtering can reduce unwanted ping-based traffic while allowing other services to remain operational.
- Verification should include both the ACL configuration and actual connectivity/service tests.