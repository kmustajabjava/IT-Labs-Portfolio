# Cisco Packet Tracer — Server-Based AAA Authentication with TACACS+ and RADIUS

## Overview

This lab focused on configuring and verifying **server-based AAA (Authentication, Authorization, and Accounting) authentication** using two different AAA protocols:

- **TACACS+** on router R2
- **RADIUS** on router R3

The lab also demonstrated the use of a **local username database as a backup authentication method** when the external AAA server is unavailable.

The configurations were tested through the router console, and the completed activity was verified using Cisco Packet Tracer's **Check Results** feature.

---

## Objectives

- Configure server-based AAA authentication using **TACACS+**.
- Verify TACACS+ authentication from the **R2 console**.
- Configure server-based AAA authentication using **RADIUS**.
- Verify RADIUS authentication from the **R3 console**.
- Configure local authentication as a fallback mechanism.
- Understand the difference between TACACS+ and RADIUS authentication.
- Verify the completed configuration using Packet Tracer's assessment system.

---

## Lab Environment

**Platform:** Cisco Packet Tracer

**Topology:** R1, R2, R3 with PC clients and dedicated AAA servers.

**AAA protocols used:**

| Router | AAA Protocol | AAA Server |
|---|---|---|
| R2 | TACACS+ | `192.168.2.2` |
| R3 | RADIUS | `192.168.3.2` |

---

# Part 1 — Server-Based AAA Authentication Using TACACS+ on R2

## Step 1 — Test Network Connectivity

Before configuring AAA, connectivity between the network devices was verified by testing communication between:

- PC-A and PC-B
- PC-A and PC-C
- PC-B and PC-C

Successful connectivity confirmed that the network was ready for AAA configuration.

---

## Step 2 — Configure a Local Backup Account on R2

A local user account was created on R2 to provide a fallback authentication method if the TACACS+ server becomes unavailable.

The following configuration was used:

```text
enable
configure terminal
username Admin2 secret admin2pa55
```

The local database therefore contains:

```text
Username: Admin2
Password: admin2pa55
```

### Purpose

The local account provides a backup authentication mechanism.

The final AAA authentication method was configured to try the TACACS+ server first and use the local database if the TACACS+ server could not be reached.

---

# Step 3 — Verify the TACACS+ Server

The pre-configured TACACS+ server was opened in Cisco Packet Tracer.

Under:

```text
Services → AAA
```

the following configuration was verified:

- Network client: **R2**
- TACACS+ shared secret: configured by the lab
- User account: **Admin2**

The server was already prepared for R2 authentication.

---

# Step 4 — Configure TACACS+ on R2

The TACACS+ server address and shared secret were configured on R2.

```text
tacacs-server host 192.168.2.2
tacacs-server key tacacspa55
```

This configured R2 to communicate with the TACACS+ server at:

```text
192.168.2.2
```

The shared key used between R2 and the TACACS+ server was:

```text
tacacspa55
```

> These commands are used because Cisco Packet Tracer's IOS implementation for this lab does not support the newer `tacacs server` syntax.

---

# Step 5 — Enable AAA Authentication on R2

AAA was enabled on R2:

```text
aaa new-model
```

The default login authentication method was then configured:

```text
aaa authentication login default group tacacs+ local
```

### Authentication flow

The configuration instructs R2 to:

1. Contact the TACACS+ server first.
2. Authenticate the user through TACACS+.
3. If the TACACS+ server is unavailable, use R2's local username database.

Conceptually:

```text
Console Login
      ↓
     AAA
      ↓
 TACACS+ Server
      ↓
 Available?
   ↙       ↘
 Yes        No
  ↓          ↓
TACACS+    Local DB
             ↓
           Admin2
```

This provides both centralized authentication and a local fallback mechanism.

---

# Step 6 — Configure Console Authentication

The R2 console was configured to use the default AAA authentication method:

```text
line console 0
login authentication default
```

This associates the console login process with the AAA method named `default`.

The `default` method was previously configured as:

```text
group tacacs+ local
```

---

# Step 7 — Verify TACACS+ Authentication

The R2 console login was tested after the AAA configuration.

The TACACS+ credentials supplied by the lab were used:

```text
Username: Admin2
Password: admin2pa55
```

The login was successful, confirming that R2 could authenticate the user through the configured TACACS+ authentication system.

---

# Part 2 — Server-Based AAA Authentication Using RADIUS on R3

## Step 1 — Configure a Local Backup Account on R3

A local user account was created on R3 for backup authentication:

```text
enable
configure terminal
username Admin3 secret admin3pa55
```

The local database therefore contains:

```text
Username: Admin3
Password: admin3pa55
```

This account provides a fallback authentication method if the RADIUS server becomes unavailable.

---

# Step 2 — Verify the RADIUS Server

The pre-configured RADIUS server was opened in Cisco Packet Tracer.

Under:

```text
Services → AAA
```

the following configuration was verified:

- Network client: **R3**
- RADIUS shared secret: configured by the lab
- User account: **Admin3**

---

# Step 3 — Configure RADIUS on R3

R3 was configured with the RADIUS server address and shared secret:

```text
radius-server host 192.168.3.2
radius-server key radiuspa55
```

The RADIUS server address was:

```text
192.168.3.2
```

The shared secret was:

```text
radiuspa55
```

> Cisco Packet Tracer uses the `radius-server` commands for this lab because the newer `radius server` syntax is not supported by the IOS version used in the activity.

---

# Step 4 — Enable AAA Authentication on R3

AAA was enabled:

```text
aaa new-model
```

The default login authentication method was configured to use RADIUS first and the local database as a fallback:

```text
aaa authentication login default group radius local
```

### Authentication flow

```text
Console Login
      ↓
     AAA
      ↓
 RADIUS Server
      ↓
 Available?
   ↙       ↘
 Yes        No
  ↓          ↓
RADIUS     Local DB
             ↓
           Admin3
```

---

# Step 5 — Configure Console Authentication on R3

The console was configured to use the default AAA authentication method:

```text
line console 0
login authentication default
```

This causes console authentication to follow the RADIUS-first authentication policy.

---

# Step 6 — Verify RADIUS Authentication

The R3 console login was tested using the RADIUS account provided by the lab:

```text
Username: Admin3
Password: admin3pa55
```

The authentication was successful, confirming that R3 was able to use the configured RADIUS authentication service.

---

# Step 7 — Check Results

The completed configuration was verified using Cisco Packet Tracer's **Check Results** feature.

The activity achieved:

**100% completion**

This confirmed that the required TACACS+, RADIUS, AAA, local fallback, console authentication, and verification components were correctly configured.

---

# TACACS+ vs RADIUS

This lab provided practical experience with two commonly used AAA protocols.

| Feature | TACACS+ | RADIUS |
|---|---|---|
| Router | R2 | R3 |
| Server | `192.168.2.2` | `192.168.3.2` |
| Authentication method | `group tacacs+ local` | `group radius local` |
| Local fallback | Yes | Yes |
| Primary purpose in this lab | Console authentication | Console authentication |

Both protocols allowed the router to use a centralized authentication server rather than relying exclusively on locally configured credentials.

---

# Local Fallback Authentication

An important part of this lab was configuring a local database as a backup.

### R2

```text
username Admin2 secret admin2pa55
aaa authentication login default group tacacs+ local
```

### R3

```text
username Admin3 secret admin3pa55
aaa authentication login default group radius local
```

The `local` keyword provides a fallback method if the external AAA server is unavailable.

This is important because losing access to a centralized authentication server should not necessarily lock administrators out of network devices.

---

# Key Security Concepts Learned

### Authentication

Authentication verifies the identity of a user before allowing access to the router.

In this lab, authentication was performed using:

- TACACS+
- RADIUS
- Local username databases

### Authorization

AAA can also be used to control what authenticated users are permitted to do. This lab primarily focused on authentication rather than detailed command authorization.

### Centralized Authentication

Using TACACS+ or RADIUS allows authentication to be managed through a centralized server rather than creating independent administrator credentials on every router.

### Backup Authentication

The local database provides a fallback mechanism when the external AAA server is unavailable.

### Secure Shared Secrets

TACACS+ and RADIUS require a shared secret between the network device and AAA server to establish trusted communication.

---

# Important Configuration Commands

## TACACS+ — R2

```text
tacacs-server host 192.168.2.2
tacacs-server key tacacspa55
aaa new-model
aaa authentication login default group tacacs+ local
line console 0
login authentication default
```

## RADIUS — R3

```text
radius-server host 192.168.3.2
radius-server key radiuspa55
aaa new-model
aaa authentication login default group radius local
line console 0
login authentication default
```

## Local Backup Users

### R2

```text
username Admin2 secret admin2pa55
```

### R3

```text
username Admin3 secret admin3pa55
```

---

# Skills Demonstrated

- Cisco IOS CLI
- AAA configuration
- TACACS+ authentication
- RADIUS authentication
- Local user database configuration
- Console authentication
- Centralized authentication
- Authentication fallback mechanisms
- Network connectivity testing
- Cisco Packet Tracer troubleshooting
- Basic network security administration

---

# Files

Recommended GitHub structure:

```text
AAA-TACACS-RADIUS/
├── README.md
├── AAA-TACACS-RADIUS.pkt
└── screenshots/
    └── assessment-100-percent.png
```

If you have screenshots of the successful R2 and R3 login tests, they can also be added:

```text
screenshots/
├── network.png
├── R2-config-1.png
└── R3-config-1.png


```

---

## Conclusion

This lab provided hands-on experience configuring **server-based AAA authentication** using both TACACS+ and RADIUS.

R2 was configured to authenticate administrators through a TACACS+ server, while R3 was configured to use a RADIUS server. Both routers also maintained local accounts as fallback authentication methods.

The configurations were successfully tested through the router consoles, and the completed Packet Tracer activity achieved **100% assessment completion**.

This exercise strengthened practical understanding of centralized authentication, AAA configuration, console security, fallback authentication, and basic network access control.