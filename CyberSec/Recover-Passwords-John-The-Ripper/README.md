# Lab — Recover Passwords

## Objective

* Use **John the Ripper** to recover weak user passwords from password hashes.
* Change a user's password to a stronger password.
* Demonstrate how a dictionary-based password attack behaves against a stronger password.

---

## Lab Environment

* **VM:** Cisco CSE-LABVM
* **Operating System:** Linux
* **Tool:** John the Ripper
* **Password database:** `/etc/passwd` and `/etc/shadow`

> **Note:** Cisco's original lab documentation shows five password hashes. The CSE-LABVM used for this investigation contained **seven password hashes**, so the results below reflect the actual VM rather than the example output in the Cisco instructions.

---

# Part 1 — Access John the Ripper

Open a terminal in the CSE-LABVM and navigate to the John the Ripper directory:

```bash
cd ~/Downloads/john/run
```

Verify that John is available:

```bash
./john
```

John the Ripper is an open-source password recovery tool that can test password hashes against dictionaries and password-generation rules.

---

# Part 2 — Combine User and Password Information

Linux stores different parts of account information in two files:

```text
/etc/passwd
/etc/shadow
```

The `/etc/passwd` file contains information such as usernames, user IDs, home directories, and login shells.

The `/etc/shadow` file contains password hashes and other password-related information and normally requires elevated privileges to access.

John needs the relevant account and hash information together, so the two files are combined using `unshadow`.

Run:

```bash
sudo ./unshadow /etc/passwd /etc/shadow > mypasswd
```

When prompted for the `cisco` user's sudo password, enter the password configured for the lab VM.

### Why use `unshadow`?

`unshadow` combines the account information from `/etc/passwd` with the password hashes from `/etc/shadow` into a format that John the Ripper can process.

The resulting file is:

```text
mypasswd
```

---

# Part 3 — Check the Password Hashes

Before attempting to recover the passwords, use:

```bash
./john --show mypasswd
```

The Cisco documentation originally expects:

```text
0 password hashes cracked, 5 left
```

However, the VM used in this lab contained:

```text
7 password hashes
```

This difference is due to the account configuration/version of the lab VM.

### Finding

John detected seven password hashes that could be processed.

### What this indicates

The password database was successfully extracted and converted into a format that John could use for password recovery.

No additional users needed to be created because the lab VM already contained password-bearing accounts.

---

# Part 4 — Recover the Passwords

The lab provides a password dictionary called:

```text
password.lst
```

John can use this dictionary together with its rules to generate password variations and compare them against the hashes.

Run:

```bash
./john --wordlist=password.lst --rules mypasswd --format=crypt
```

John then attempts to recover the passwords using the supplied dictionary and rules.

### Result

John was able to recover the passwords represented by the hashes in the lab VM.

The number of recovered passwords may differ from Cisco's example because the VM contained seven hashes rather than five.

To display the recovered passwords:

```bash
./john --show mypasswd
```

### Finding

The weak passwords were recoverable using a dictionary-based attack.

### What this indicates

Passwords that are common, short, predictable, or included in password dictionaries can potentially be recovered very quickly when an attacker obtains the corresponding password hashes.

This demonstrates why simply storing a password as a hash does not make a weak password secure.

---

# Part 5 — Change a User's Password

The next part demonstrates how a stronger password changes the result of the same attack.

Cisco's lab uses the `Eric` account for this demonstration.

Change Eric's password:

```bash
sudo passwd Eric
```

Enter a strong password when prompted:

```text
New password:
Retype new password:
```

A successful change should produce:

```text
passwd: password updated successfully
```

### Finding

Eric's original weak password has now been replaced with a stronger password.

### Security principle

A strong password should be:

* Long
* Unique
* Difficult to guess
* Not based on common words or predictable patterns
* Not reused across different services

---

# Part 6 — Recreate the Password File

Because Eric's password has changed, the `/etc/shadow` file has also changed.

The `mypasswd` file must therefore be regenerated:

```bash
sudo ./unshadow /etc/passwd /etc/shadow > mypasswd
```

This ensures that John is working with the current password hashes.

---

# Part 7 — Attempt to Recover the Strong Password

Run the same John command again:

```bash
./john --wordlist=password.lst --rules mypasswd --format=crypt
```

John should identify the hashes that remain unrecovered.

Depending on the other accounts in the VM, the output may contain several previously recovered passwords and one or more remaining hashes.

Allow John to run for a while.

To stop the session:

```text
q
```

or:

```text
Ctrl+C
```

Then check the results:

```bash
./john --show mypasswd
```

### Expected observation

The weak passwords that are present in the dictionary/rules can be recovered, while the new strong password should remain unrecovered by this particular dictionary attack.

---

# Part 8 — Analysis

The lab demonstrates two important concepts.

## Weak Password

```text
Weak password
      ↓
Included in password dictionary/rules
      ↓
Hash tested
      ↓
Password recovered
```

A short or commonly used password can therefore be vulnerable to dictionary-based attacks.

## Strong Password

```text
Strong unique password
      ↓
Not found in the supplied dictionary/rules
      ↓
John cannot recover it using this attack
```

A stronger password significantly increases the difficulty of password recovery, although no password should be considered absolutely impossible to crack.

---

# Security Lessons

### 1. Password hashes should be protected

An attacker who obtains password hashes can perform offline password-guessing attacks without needing to interact with the login system.

### 2. Weak passwords are vulnerable

Common passwords such as simple numbers or dictionary words can be recovered rapidly using password dictionaries and rules.

### 3. Password length matters

Longer passwords or passphrases generally provide a much larger search space than short predictable passwords.

### 4. Password uniqueness matters

A password should not be reused between different accounts or services.

### 5. Password changes update the stored hash

After changing a password, `/etc/shadow` contains a new password hash. The `mypasswd` file therefore needs to be regenerated before testing the new password.

---

# Commands Used

```bash
cd ~/Downloads/john/run
```

```bash
sudo ./unshadow /etc/passwd /etc/shadow > mypasswd
```

```bash
./john --show mypasswd
```

```bash
./john --wordlist=password.lst --rules mypasswd --format=crypt
```

```bash
sudo passwd Eric
```

```bash
sudo ./unshadow /etc/passwd /etc/shadow > mypasswd
```

```bash
./john --show mypasswd
```

---

# Conclusion

This lab demonstrated how John the Ripper can be used to recover weak Linux account passwords from password hashes.

The initial password-recovery attempt successfully recovered passwords from the lab VM's password hashes. The VM contained **seven password hashes**, whereas the original Cisco documentation demonstrates the procedure with five.

After changing Eric's password to a stronger value, the password database was regenerated and the same John the Ripper dictionary attack was performed again. The stronger password was not recovered by the supplied dictionary/rules during the lab.

The exercise demonstrates why organizations should enforce strong, unique passwords and protect password hashes from unauthorized access.
