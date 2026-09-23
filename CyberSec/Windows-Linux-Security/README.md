# Configure Security Features in Windows and Linux

Hands-on cybersecurity lab completed as part of Cisco Networking Academy coursework. The lab focused on applying basic security configuration and system-hardening practices across Windows and Linux environments.

## Objectives

* Update Windows and Linux systems
* Configure Windows local security policies
* Configure password and account lockout policies
* Review Windows audit policy
* Configure Windows Defender Firewall rules
* Install and use Linux security tools
* Perform basic rootkit and security auditing checks

## Environment

* Windows 10
* Linux virtual machine (CSE-LABVM)
* VirtualBox
* Internet connectivity

## Activities

### 1. Windows and Linux Updates

* Verified network connectivity between the Windows host and Linux virtual machine.
* Checked Windows Update configuration and update status.
* Updated Linux package indexes using `apt-get update`.
* Upgraded installed Linux packages using `apt-get upgrade`.

### 2. Windows Local Security Policy

Configured and reviewed:

* Password history
* Minimum password length
* Password expiration
* Password complexity requirements
* Account lockout threshold
* Account lockout reset period
* Audit account logon events

Tested the configured password policy using an invalid password to verify that the security requirements were enforced.

### 3. Windows Defender Firewall

* Reviewed Windows Defender Firewall configuration and allowed applications.
* Created a custom inbound firewall rule for ICMPv4 Echo Requests.
* Restricted the rule to appropriate network profiles.
* Verified connectivity between the Linux VM and Windows host after applying the rule.

### 4. Linux Security Tools

Installed and executed **chkrootkit** to check the local Linux system for indicators associated with rootkits.

Also installed **Lynis**, a Linux security auditing tool, and verified its installed version.

Example commands:

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt install chkrootkit
sudo chkrootkit
sudo apt install lynis
lynis show version
```

## Key Security Concepts

* System patching reduces exposure to known vulnerabilities.
* Password and account-lockout policies help protect user accounts against weak credentials and repeated login attempts.
* Audit policies provide security-related event information for monitoring and investigation.
* Firewalls control network traffic according to defined rules.
* ICMP traffic can be selectively allowed or restricted through firewall rules.
* Security auditing tools such as chkrootkit and Lynis can help identify potential security issues and hardening opportunities.

## Security Relevance

This lab provided practical exposure to endpoint security and basic system-hardening activities across Windows and Linux. It demonstrated how security policies, patch management, firewall configuration, and security auditing tools can be used together to reduce system exposure and improve endpoint security.
