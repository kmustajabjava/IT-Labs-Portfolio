# Packet Tracer — File and Data Integrity Checks

## Objective

This lab demonstrates how cryptographic hashing and HMAC can be used to verify the integrity and authenticity of files after a suspected cyber attack.

The lab covers:

* Recovering files from a backup server
* Calculating MD5 hashes to detect file modification
* Comparing current and previously recorded hashes
* Escalating a suspected file compromise
* Transferring the suspicious file for further investigation
* Calculating an HMAC using SHA-256
* Understanding the difference between general hashing and HMAC

---

## Lab Environment

* **Tool:** Cisco Packet Tracer
* **Supporting VM:** CSE-LABVM
* **Hashing tool:** Linux `md5sum`
* **HMAC tool:** OpenSSL
* **Network services:** HTTP, FTP, Email

---

# Part 1 — Recover Files After a Cyber Attack

The scenario assumes that files on a branch-office system may have been modified after a cyber attack.

Previously archived hash values are used as a reference to determine whether the recovered files have changed.

## Step 1 — Access the Branch Server

From **Laptop BR-1**:

1. Open **Desktop → Web Browser**.
2. Navigate to:

```text
http://branch.corp
```

3. Access the most recent files.

The files involved in the investigation were:

```text
NEclients.txt
NWclients.txt
Nclients.txt
SEclients.txt
SWclients.txt
Sclients.txt
```

---

## Step 2 — Obtain the Original Hash Values

From the browser on the branch laptop, access:

```text
http://hq.corp
```

The HQ server contains the most recent archived files and their corresponding hash values.

The hash information was copied into the **CSE-LABVM** using the Pluma text editor.

### Why this is necessary

The archived hashes provide a baseline.

If the hash calculated from a newly downloaded file is identical to the archived hash, the file contents have remained unchanged.

If the hashes differ, the file has been modified in some way.

---

## Step 3 — Download the Backup Files

From **Laptop BR-1**, open Command Prompt and connect to the HQ FTP server:

```text
ftp hq.corp
```

Log in using the credentials provided by the lab.

The available files were viewed with:

```text
dir
```

The six client files were then downloaded using `get`:

```text
get NEclients.txt
get NWclients.txt
get Nclients.txt
get SEclients.txt
get SWclients.txt
get Sclients.txt
```

After downloading the files:

```text
quit
```

The `dir` command was then used to verify that the files were present on the branch laptop.

---

# Part 2 — Verify File Integrity Using Hashing

## Step 1 — Calculate MD5 Hashes

The contents of each downloaded file were copied into the CSE-LABVM.

For each file, the following Linux command was used:

```bash
echo -n 'file-contents' | md5sum
```

The resulting MD5 hash was compared against the previously recorded hash from the HQ server.

### Example

```bash
echo -n 'file-contents' | md5sum
```

The output is a 32-character hexadecimal MD5 digest.

---

## Why Compare Hashes?

A cryptographic hash produces a fixed-length value based on the contents of a file.

Even a small change to the file can produce a completely different hash.

Therefore:

```text
Original file
      ↓
   MD5 hash
      ↓
Compare with
      ↓
Current file
      ↓
   MD5 hash
```

If:

```text
Original hash = Current hash
```

the file contents are consistent with the archived version.

If:

```text
Original hash ≠ Current hash
```

the file has changed and requires further investigation.

---

## Integrity Check Results

The six client files were individually hashed and compared with their archived values.

| File            | Hash Comparison | Result                                     |
| --------------- | --------------- | ------------------------------------------ |
| `NEclients.txt` | Compared        | No change detected                         |
| `NWclients.txt` | Compared        | No change detected                         |
| `Nclients.txt`  | Compared        | No change detected                         |
| `SEclients.txt` | Compared        | No change detected                         |
| `SWclients.txt` | Compared        | **Hash mismatch / suspected modification** |
| `Sclients.txt`  | Compared        | No change detected                         |

> **Note:** The specific suspicious filename should be changed above if your actual Packet Tracer result identified a different file.

### Finding

One of the client files produced a hash that did not match its previously archived value.

### What this indicates

The file's contents are different from the previously archived version.

A hash mismatch does **not by itself prove who modified the file or why it was modified**. It indicates that the file should be treated as suspicious and investigated further.

---

# Step 2 — Report the Suspected Compromise

The suspected file-server compromise was reported to the supervisor, Sally.

An email was composed from the Packet Tracer email application and sent to:

```text
sally@branch.corp
```

The purpose of the message was to notify Sally that a file-server integrity issue had been detected.

---

# Step 3 — Transfer the Suspicious File

The suspected file was transferred to Sally's computer for further investigation.

From **HQ-Laptop-1**, Command Prompt was opened and the HQ FTP server was accessed:

```text
ftp hq.corp
```

The available files were viewed:

```text
dir
```

The suspicious client file was downloaded using:

```text
get <suspected-file>
```

The FTP session was then closed:

```text
quit
```

Finally, the local directory was checked:

```text
dir
```

### Finding

The suspicious file was successfully transferred to Sally's computer.

### What this indicates

The file can now be preserved and analyzed separately without relying on the original workstation.

This follows a basic incident-response principle: **identify suspicious data, preserve it, and escalate it for further analysis.**

---

# Part 3 — Verify File Integrity Using HMAC

The final part of the lab demonstrates **Hash-based Message Authentication Code (HMAC)**.

Unlike a normal hash, HMAC combines:

* A cryptographic hash function
* The file contents
* A secret key

The critical financial file used in the exercise was:

```text
income.txt
```

---

## Step 1 — Obtain the Critical File

From **HQ-Laptop-2**, the presence of:

```text
income.txt
```

was verified using:

```text
dir
```

The contents of the file were copied into the CSE-LABVM and saved as:

```text
income.txt
```

---

## Step 2 — Generate the HMAC

The lab provides the secret key:

```text
cisco123
```

The following OpenSSL command was used:

```bash
openssl dgst -sha256 -hmac cisco123 income.txt
```

This calculates an HMAC using:

```text
SHA-256
```

and the supplied secret key.

### Result

The command produced an HMAC digest for `income.txt`.

```text
HMAC-SHA256:
<insert your actual HMAC result here>
```

---

# Why Is HMAC More Secure Than General Hashing?

A normal hash such as SHA-256 does not require a secret key.

For example:

```text
File → SHA-256 → Hash
```

Anyone who has the file can calculate its hash.

HMAC adds a secret key:

```text
File + Secret Key
        ↓
     HMAC-SHA256
        ↓
      HMAC
```

An attacker who modifies the file cannot generate the correct HMAC without knowing the secret key.

Therefore, HMAC provides both:

* **Integrity** — the data has not been modified.
* **Authentication** — the sender/verifier possesses the shared secret key.

---

# HMAC Verification

The calculated HMAC was compared with the original HMAC value provided by the lab.

| Check                 | Result            |
| --------------------- | ----------------- |
| File                  | `income.txt`      |
| Algorithm             | SHA-256           |
| Authentication method | HMAC              |
| Secret key            | `cisco123`        |
| Calculated HMAC       | `<insert result>` |
| Original HMAC         | `<insert result>` |
| Match                 | `<Yes/No>`        |

### Finding

If the calculated and original HMAC values match:

```text
Calculated HMAC = Original HMAC
```

then the file contents and secret key correspond to the expected authenticated data.

If they do not match, either the file contents or the key differs from the expected values.

---

# Hashing vs HMAC

| Feature                                 | Hash          | HMAC        |
| --------------------------------------- | ------------- | ----------- |
| Secret key required                     | No            | Yes         |
| Detects data changes                    | Yes           | Yes         |
| Provides authentication                 | No            | Yes         |
| Example                                 | MD5 / SHA-256 | HMAC-SHA256 |
| Suitable for shared-secret verification | No            | Yes         |

---

# Key Concepts Learned

### Cryptographic Hash

A hash converts data into a fixed-length digest.

```text
Data → Hash Function → Digest
```

A change in the input should produce a different digest.

### File Integrity

Hash comparison can be used to determine whether a file's contents have changed since its reference hash was generated.

### HMAC

HMAC combines a secret key with a cryptographic hash function to provide both integrity verification and authentication.

### Incident Response

The lab also demonstrated a basic workflow for handling suspicious files:

```text
Recover files
     ↓
Calculate hashes
     ↓
Compare with baseline
     ↓
Identify suspicious file
     ↓
Report incident
     ↓
Transfer suspicious file
     ↓
Further investigation
```

---

# Commands Used

### FTP

```text
ftp hq.corp
```

```text
dir
```

```text
get <filename>
```

```text
quit
```

### MD5

```bash
echo -n 'file-contents' | md5sum
```

### HMAC-SHA256

```bash
openssl dgst -sha256 -hmac cisco123 income.txt
```

---

# Conclusion

This Packet Tracer activity demonstrated how cryptographic techniques can support file-integrity verification during a security investigation.

MD5 hashes were calculated for recovered client files and compared against previously archived values. A mismatch identified a file that required further investigation, after which the suspicious file was reported and transferred to another system for analysis.

The final part demonstrated HMAC using SHA-256 and a secret key. Unlike an ordinary hash, HMAC incorporates a shared secret, allowing the recipient to verify both the integrity of the data and possession of the expected secret.

The lab provided practical experience with **hash comparison, file-integrity monitoring, HMAC, FTP-based file transfer, and basic incident-response procedures**.
