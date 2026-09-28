# Linux – Configure Authentication and Authorization

## Overview

This lab demonstrates fundamental Linux authentication and authorization concepts using the command line.

The activity involved creating a new user group, creating user accounts, assigning users to groups, switching between users, and controlling access to directories using Linux file permissions.

The lab also demonstrated both **symbolic** and **absolute (octal)** permission modes with `chmod`.

---

## Objectives

* Create a new Linux group.
* Create and configure user accounts.
* Add users to a group.
* Understand Linux user and group ownership.
* Switch between different user accounts.
* Inspect Linux file and directory permissions.
* Modify permissions using symbolic mode.
* Modify permissions using absolute/octal mode.
* Verify how permissions affect access to files and directories.

---

## Lab Environment

* **CSE-LABVM**
* Linux virtual machine running in VirtualBox
* Linux terminal
* Root/sudo privileges

---

# Part 1 – Create a New User Group

A new group named `HR` was created for the lab users.

### Commands Used

```bash
sudo su
```

Create the group:

```bash
groupadd HR
```

Verify the group:

```bash
cat /etc/group
```

The `HR` group was successfully added to the system.

### Concept

Linux groups allow administrators to organize users and assign permissions collectively instead of configuring access individually for every user.

---

# Part 2 – Create Users and Add Them to the Group

Two new user accounts were created:

```text
jenny
joe
```

### Create Jenny

```bash
adduser jenny
```

### Add Jenny to HR

```bash
usermod -G HR jenny
```

### Create Joe

```bash
adduser joe
```

### Add Joe to HR

```bash
usermod -G HR joe
```

The users were then verified using:

```bash
cat /etc/passwd
```

The password-related account information was also inspected through:

```bash
cat /etc/shadow
```

> Passwords and password hashes are intentionally not included in this repository.

### Concept

The lab demonstrated the difference between:

* **Authentication** — verifying who a user is.
* **Authorization** — determining what that authenticated user is allowed to access or modify.

---

# Part 3 – Switch Users and Modify Permissions

The lab then examined Linux directory permissions using different user accounts.

The `/home` directory was inspected with:

```bash
ls -l
```

A typical permission entry such as:

```text
drwxr-xr-x
```

can be divided into:

```text
d | rwx | r-x | r-x
  |     |     |
  |     |     └── Others
  |     └──────── Group
  └────────────── Owner
```

### Permission Groups

| Permission | Meaning |
| ---------- | ------- |
| `r`        | Read    |
| `w`        | Write   |
| `x`        | Execute |

For directories:

* `r` allows listing directory contents.
* `w` allows creating, deleting, or modifying entries when combined with appropriate access.
* `x` allows entering/traversing the directory.

---

## Testing Access to Joe's Directory

As Jenny, I attempted to access Joe's home directory.

Initially, the directory allowed other users to enter it because the `others` permission included execute access.

Inside Joe's directory, Jenny attempted:

```bash
touch new.txt
```

The operation was denied because Jenny did not have write permission for Joe's directory.

This demonstrated that being able to **enter a directory does not automatically mean having permission to create or modify files inside it**.

---

# Modify Permissions Using Symbolic Mode

As an administrator, the following command was used:

```bash
chmod o-x joe
```

This removed execute permission from the `others` category for Joe's directory.

The result was verified using:

```bash
ls -l
```

Jenny then attempted:

```bash
cd joe
```

The operation was denied.

### Concept

Symbolic `chmod` allows permissions to be modified using letters representing:

* `u` — user/owner
* `g` — group
* `o` — others
* `a` — all

Examples:

```bash
chmod u+rwx file
chmod u+rw file
chmod o+r file
chmod g-rwx file
```

---

# Part 4 – Modify Permissions Using Absolute Mode

Linux permissions can also be represented using **octal values**.

| Number | Permissions            |
| -----: | ---------------------- |
|    `7` | Read + Write + Execute |
|    `6` | Read + Write           |
|    `5` | Read + Execute         |
|    `4` | Read                   |
|    `3` | Write + Execute        |
|    `2` | Write                  |
|    `1` | Execute                |
|    `0` | No permissions         |

The three digits represent:

```text
Owner | Group | Others
```

For example:

```bash
chmod 764 examplefile
```

means:

```text
Owner  = 7 → rwx
Group  = 6 → rw-
Others = 4 → r--
```

---

## Modify Joe's Directory

The following command was used:

```bash
chmod 705 joe
```

This resulted in:

```text
Owner  → rwx
Group  → ---
Others → r-x
```

The permissions were verified using:

```bash
ls -l
```

---

## Create and Test a File

As Joe, a test file was created:

```bash
touch test.txt
```

The file was then verified:

```bash
ls -l
```

After switching back to Jenny, the contents of Joe's directory could be viewed because `others` had read/execute access.

However, Jenny attempted:

```bash
touch jenny.txt
```

and received:

```text
Permission denied
```

This demonstrated that Jenny could **read/access** the directory but could not **write** to it.

---

# Commands Practiced

### Switch to root

```bash
sudo su
```

### Create a group

```bash
groupadd HR
```

### Create a user

```bash
adduser jenny
```

### Add a user to a group

```bash
usermod -G HR jenny
```

### View user accounts

```bash
cat /etc/passwd
```

### View protected account information

```bash
cat /etc/shadow
```

### Display current directory

```bash
pwd
```

### Change directory

```bash
cd /home
```

### List permissions

```bash
ls -l
```

### Change permissions using symbolic mode

```bash
chmod o-x joe
```

### Change permissions using absolute mode

```bash
chmod 705 joe
```

### Create a file

```bash
touch test.txt
```

### Switch users

```bash
su cisco
```

---

# Authentication vs Authorization

One of the main concepts demonstrated by this lab was the difference between authentication and authorization.

### Authentication

Authentication answers:

> **Who are you?**

Examples in this lab included logging in as:

```text
jenny
joe
cisco
```

using their respective passwords.

### Authorization

Authorization answers:

> **What are you allowed to do?**

Linux uses ownership and permissions to control access to files and directories.

For example:

```text
Owner  → rwx
Group  → r-x
Others → r--
```

determines what each category of user can do.

---

# Key Takeaways

* Linux uses **users and groups** to manage access.
* Authentication verifies a user's identity.
* Authorization determines what resources the user can access.
* File and directory permissions are divided into **owner, group, and others**.
* `chmod` can modify permissions using symbolic or octal notation.
* Directory `x` permission controls whether a user can traverse/enter the directory.
* Write permission is required to create or modify directory entries.
* `ls -l` is useful for inspecting ownership and permissions.
* `/etc/passwd` contains account information, while `/etc/shadow` contains protected password-related data.
* Proper permission configuration helps enforce the **principle of least privilege**.

---

## Lab Outcome

**Completed successfully.**

This lab provided practical experience with:

* Linux user management
* Linux group management
* Authentication
* Authorization
* File and directory permissions
* `chmod`
* Symbolic permissions
* Octal/absolute permissions
* User switching
* Access-control testing
* Basic Linux security administration
