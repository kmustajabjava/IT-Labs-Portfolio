# Zone-Based Policy Firewall (ZPF) Configuration

## Objective

Configure and verify a Zone-Based Policy Firewall (ZPF) on Cisco Router R3 to control traffic between an internal network and an external network.

This lab demonstrates how to define security zones, classify traffic, apply inspection policies, and control traffic flows using Cisco IOS commands.

## Network Topology

The lab uses three routers and two PCs.

| Device | Interface | IP Address | Purpose |
|---|---|---|---|
| R1 | G0/1 | 192.168.1.1/24 | PC-A LAN gateway |
| R1 | S0/0/0 | 10.1.1.1/30 | Connection to R2 |
| R2 | S0/0/0 | 10.1.1.2/30 | Connection to R1 |
| R2 | S0/0/1 | 10.2.2.2/30 | Connection to R3 |
| R3 | S0/0/1 | 10.2.2.1/30 | External-facing interface |
| R3 | G0/1 | 192.168.3.1/24 | Internal LAN gateway |
| PC-A | NIC | 192.168.1.3/24 | External-side host |
| PC-C | NIC | 192.168.3.3/24 | Internal-side host |

**Firewall location:** R3

## Tools and Technologies

- Cisco Packet Tracer
- Cisco IOS CLI
- Zone-Based Policy Firewall (ZPF)
- IPv4 ACLs
- Class maps and policy maps
- Stateful traffic inspection
- SSH, HTTP, ICMP

## 1. Verify Initial Connectivity

Before configuring the firewall, connectivity was tested between the devices.

The initial checks included:

- ICMP connectivity between PC-A and PC-C
- HTTP access to the web server
- SSH access from both PCs to R2's `S0/0/1` interface (`10.2.2.2`)

Verifying baseline connectivity helps distinguish firewall-related failures from pre-existing network configuration problems.

## 2. Create Security Zones on R3

Two security zones were created:

- `PRIVATE` — internal network
- `PUBLIC` — external network

```cisco
R3(config)# zone security PRIVATE
R3(config)# zone security PUBLIC
```

## 3. Create an ACL for Internal Traffic

ACL 101 identifies IP traffic originating from the internal LAN `192.168.3.0/24`.

```cisco
R3(config)# access-list 101 permit ip 192.168.3.0 0.0.0.255 any
```

The wildcard mask `0.0.0.255` matches the entire `/24` internal subnet.

## 4. Configure the Class Map

The class map references ACL 101 to classify internal traffic for inspection.

```cisco
R3(config)# class-map type inspect match-all INTERNAL-TRAFFIC
R3(config-cmap)# match access-group 101
R3(config-cmap)# exit
```

## 5. Configure the Inspection Policy Map

The policy map defines the action applied to traffic matching `INTERNAL-TRAFFIC`.

```cisco
R3(config)# policy-map type inspect PRIVATE-TO-PUBLIC
R3(config-pmap)# class type inspect INTERNAL-TRAFFIC
R3(config-pmap-c)# inspect
R3(config-pmap-c)# exit
R3(config-pmap)# exit
```

The `inspect` action enables stateful inspection, allowing return traffic for permitted sessions without requiring a separate policy for every response.

## 6. Create the Zone Pair

The zone pair defines the direction of the firewall policy: from the internal `PRIVATE` zone to the external `PUBLIC` zone.

```cisco
R3(config)# zone-pair security PRIVATE-TO-PUBLIC source PRIVATE destination PUBLIC
```

Attach the inspection policy:

```cisco
R3(config-sec-zone-pair)# service-policy type inspect PRIVATE-TO-PUBLIC
```

## 7. Assign Interfaces to Security Zones

The internal and external interfaces were assigned to their respective zones.

### Internal interface

```cisco
R3(config)# interface gigabitEthernet 0/1
R3(config-if)# zone-member security PRIVATE
R3(config-if)# exit
```

### External interface

```cisco
R3(config)# interface serial 0/0/1
R3(config-if)# zone-member security PUBLIC
R3(config-if)# exit
```

## 8. Verification Commands

The following commands can be used to verify the configuration and inspect active sessions.

```cisco
R3# show zone security
R3# show zone-pair security
R3# show class-map type inspect
R3# show policy-map type inspect
R3# show policy-map type inspect zone-pair sessions
R3# show access-lists
```

The zone-pair session command displays active inspected sessions, including available protocol, source, destination, and port information.

## 9. Connectivity and Firewall Testing

The following tests were used to evaluate the firewall policy.

| Test | Expected Result | Explanation |
|---|---|---|
| PC-C → R2 SSH (`10.2.2.2`) | Allowed | Internal traffic matches ACL 101 and is inspected |
| PC-C → PC-A web server (`192.168.1.3`) | Allowed, if routing and HTTP service are operational | Internal-to-external traffic is inspected |
| PC-A → PC-C ping (`192.168.3.3`) | Blocked | New traffic initiated from the external zone is not permitted |
| R2 → PC-C ping (`192.168.3.3`) | Blocked | New traffic initiated from the external side is not permitted |

**Note:** These are expected results based on the configured policy. Actual test results should be recorded separately if they differ. ICMP uses no TCP or UDP port numbers.

## Security Concepts Demonstrated

### Stateful Inspection

The firewall tracks permitted sessions and allows associated return traffic.

### Zone-Based Access Control

Interfaces are assigned to security zones, and traffic between zones is governed by zone-pair policies.

### Least-Privilege Network Access

The policy permits inspected internal-to-external traffic while preventing unsolicited traffic from entering the protected network.

### ACL-Based Traffic Classification

An extended IPv4 ACL identifies traffic from the internal subnet, and a class map uses that ACL to select traffic for inspection.

## Key Takeaways

- Verify connectivity before deploying firewall policies.
- Define security zones and assign the correct interfaces.
- Use ACLs and class maps to classify traffic.
- Use policy maps to specify inspection actions.
- Configure zone pairs to define permitted traffic directions.
- Stateful inspection allows legitimate return traffic for inspected sessions.
- Use session and policy-map commands to troubleshoot firewall behavior.

## Lab Environment

**Platform:** Cisco Packet Tracer  
**Device:** Cisco IOS router R3  
**Firewall type:** Zone-Based Policy Firewall (ZPF)  
**Focus:** Stateful inspection and internal-to-external traffic control

*Credentials from the original lab activity are intentionally excluded from this repository.*
