# Standard ACL Planning and Implementation

## Objective

Configure, apply, and verify **numbered standard IPv4 ACLs** on multiple routers to enforce network access policies.

## Scenario

The network uses EIGRP for routing and has the following security requirements:

### R2

The `192.168.11.0/24` network must not access the Web Server at `192.168.20.254`.

All other traffic should be permitted.

### R3

The `192.168.10.0/24` network must not communicate with the `192.168.30.0/24` network.

All other traffic should be permitted.

## Part 1 — ACL Planning

Before configuring the ACLs, network connectivity was verified to ensure that the network was functioning normally.

The ACLs were planned close to the destination networks:

- R2 → Outbound toward `192.168.20.0/24`
- R3 → Outbound toward `192.168.30.0/24`

## Part 2 — Configure ACL on R2

### 1. Create the ACL

```text
R2(config)# access-list 1 deny 192.168.11.0 0.0.0.255
R2(config)# access-list 1 permit any
```

The first statement denies traffic originating from `192.168.11.0/24`.

The second statement allows all other traffic.

### 2. Verify

```text
R2# show access-lists
```

Expected:

```text
Standard IP access list 1
    10 deny 192.168.11.0 0.0.0.255
    20 permit any
```

### 3. Apply the ACL

```text
R2(config)# interface gigabitEthernet 0/0
R2(config-if)# ip access-group 1 out
```

## Configure ACL on R3

### 1. Create the ACL

```text
R3(config)# access-list 1 deny 192.168.10.0 0.0.0.255
R3(config)# access-list 1 permit any
```

### 2. Verify

```text
R3# show access-lists
```

Expected:

```text
Standard IP access list 1
    10 deny 192.168.10.0 0.0.0.255
    20 permit any
```

### 3. Apply the ACL

```text
R3(config)# interface gigabitEthernet 0/0
R3(config-if)# ip access-group 1 out
```

## Verification Tests

The following traffic should be observed:

| Source | Destination | Expected |
|---|---|---|
| 192.168.10.10 | 192.168.11.10 | Allowed |
| 192.168.10.10 | 192.168.20.254 | Allowed |
| 192.168.11.10 | 192.168.20.254 | **Blocked** |
| 192.168.10.10 | 192.168.30.10 | **Blocked** |
| 192.168.11.10 | 192.168.30.10 | Allowed |
| 192.168.30.10 | 192.168.20.254 | Allowed |

ACL counters were checked using:

```text
R2# show access-lists
R3# show access-lists
```

## Security Concept

This lab demonstrates how standard ACLs can enforce **network-level segmentation** by blocking traffic from specific source networks while allowing all other traffic.

## Key Takeaways

- Standard ACLs use source IP addresses for filtering.
- `permit any` is required when other traffic should remain allowed.
- ACLs must be applied to an interface and direction to become effective.
- Testing before and after ACL deployment helps identify unintended connectivity problems.
- ACL match counters provide useful evidence that the rules are being triggered.