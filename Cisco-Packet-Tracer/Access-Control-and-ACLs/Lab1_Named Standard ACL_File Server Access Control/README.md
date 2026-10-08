# Named Standard ACL — File Server Access Control

## Objective

Configure and verify a **named standard IPv4 ACL** to restrict access to a file server while allowing only authorized hosts to access it.

## Scenario

The network contains a Web Server, File Server, and three workstations.

The File Server contains the database used by web applications. The security policy requires:

- **PC1 (Web Manager workstation)** → Allowed to access the File Server
- **Web Server** → Allowed to access the File Server
- **All other hosts** → Denied access to the File Server
- Access to the Web Server remains available to all workstations

## ACL Configuration

A named standard ACL was configured on **R1**.

### 1. Create the named ACL

```text
R1(config)# ip access-list standard File_Server_Restrictions
R1(config-std-nacl)# permit host 192.168.20.4
R1(config-std-nacl)# permit host 192.168.100.100
R1(config-std-nacl)# deny any
```

The ACL uses source IP addresses to determine which hosts are permitted to access the File Server.

### 2. Verify the ACL

```text
R1# show access-lists
```

Expected configuration:

```text
Standard IP access list File_Server_Restrictions
    10 permit host 192.168.20.4
    20 permit host 192.168.100.100
    30 deny any
```

### 3. Apply the ACL

The ACL was applied **outbound** on R1's FastEthernet 0/1 interface:

```text
R1(config)# interface fastethernet 0/1
R1(config-if)# ip access-group File_Server_Restrictions out
```

## Verification

The ACL was verified using:

```text
R1# show access-lists
R1# show running-config
```

or:

```text
R1# show ip interface fastethernet 0/1
```

### Connectivity tests

Expected results:

| Source | Web Server | File Server |
|---|---|---|
| PC1 | Allowed | Allowed |
| PC2 | Allowed | Denied |
| PC3 | Allowed | Denied |
| Web Server | — | Allowed |

The ACL counters can also be checked with:

```text
R1# show access-lists
```

to determine how many packets matched each ACL statement.

## Security Concept

This lab demonstrates how a **standard ACL** can restrict access based on the **source IP address**. It also demonstrates ACL placement and the importance of ordering ACL statements correctly.

## Key Takeaways

- Standard ACLs filter traffic based primarily on the source address.
- ACL statements are processed from top to bottom.
- A `deny any` statement explicitly blocks all remaining sources.
- ACL placement and direction determine which traffic is filtered.
- ACL counters can be used to verify that filtering rules are being matched.