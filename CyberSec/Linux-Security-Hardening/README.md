# Harden a Linux System

Hands-on Linux security hardening lab completed through Cisco Networking Academy.

## Objectives

* Use Lynis to audit a Linux system for security issues.
* Review warnings and hardening recommendations.
* Implement a security improvement and verify the result.

## Environment

* Ubuntu Linux VM
* VMware Workstation 17
* Lynis

## Steps

### 1. Install and Update Lynis

* Checked the installed Lynis version.
* Added the CISOfy repository.
* Updated package information.
* Installed/updated Lynis.
* Verified the installed version.

### 2. Check Lynis Updates

```bash
sudo lynis update info
```

Verified the update status of the installed Lynis version.

### 3. Run Security Audit

```bash
sudo lynis --auditor cisco
```

Used Lynis to audit system configuration, services, installed software, firewall configuration, and other security areas.

### 4. Review Results

Reviewed the **Warnings** and **Suggestions** reported by Lynis and identified a security issue requiring attention.

### 5. Apply Hardening

* Researched the identified warning.
* Implemented an appropriate security configuration.
* Ran the Lynis audit again to verify the change.

## Key Concepts

Linux Security Auditing • Vulnerability Assessment • System Hardening • Security Configuration • Audit Verification

## Tools

**Lynis • Ubuntu Linux • VMware Workstation 17 • Linux CLI**
