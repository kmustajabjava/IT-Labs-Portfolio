# Cisco Packet Tracer – Explore File and Data Encryption

## Overview

This lab demonstrates how encrypted files can be transferred through an FTP server and later decrypted by an authorized user.

The activity uses **Cisco Packet Tracer**, a **CSE-LABVM Linux virtual machine**, **FTP**, and **OpenSSL** to explore:

* AES-256-CBC encryption and decryption
* Base64-encoded encrypted data
* PBKDF2-based password derivation
* FTP authentication and file transfer
* Secure handling of encrypted files
* Security limitations of FTP and AES-CBC

> **Note:** This activity is intended for learning purposes. The encryption approach used by the lab has security limitations and should not be used to protect genuinely sensitive information in real-world environments.

---

## Objectives

* Decrypt Mary's encrypted FTP credentials.
* Connect to the Branch Office FTP server.
* Upload an encrypted confidential file.
* Decrypt Bob's encrypted FTP credentials.
* Download the encrypted file using FTP.
* Retrieve the file's decryption key.
* Decrypt the confidential file using OpenSSL.
* Understand the security limitations of FTP and unauthenticated encryption.

---

## Lab Environment

* **Cisco Packet Tracer**
* **CSE-LABVM** Linux virtual machine
* Branch Office network
* Laptop BR-1 — Mary's workstation
* Laptop BR-2 — Bob's workstation
* BR Server — FTP server
* OpenSSL
* FTP

---

# Part 1 – Discover Mary's FTP Credentials

Mary's FTP credentials were stored in an encrypted text file:

```text
maryftplogin.txt
```

The encrypted content was copied from Laptop BR-1 to the CSE-LABVM.

The following OpenSSL command was used to decrypt the Base64-encoded ciphertext:

```bash
echo '<encrypted-data>' | openssl aes-256-cbc -pbkdf2 -a -d
```

The decryption password supplied by the lab was entered when prompted.

### Result

Mary's FTP account credentials were successfully recovered and used to authenticate to the Branch Office FTP server.

> The recovered username and password are intentionally not published in this repository.

---

# Part 2 – Upload Confidential Data Using FTP

The file:

```text
clientinfo.enc
```

was opened on Mary's laptop.

The contents were already **encrypted**, rather than being stored as readable plaintext.

I then connected to the Branch Server using FTP:

```text
ftp 10.0.3.30
```

After authenticating with Mary's recovered FTP credentials, I checked the server contents:

```text
dir
```

The encrypted file was uploaded using:

```text
put clientinfo.enc
```

The `dir` command was used again to verify that the file was successfully uploaded.

The FTP session was then closed:

```text
quit
```

### Security Observation

Although `clientinfo.enc` was encrypted, **FTP itself does not encrypt the communication channel**.

Therefore, an attacker monitoring the connection could potentially observe FTP commands, authentication information, and other unencrypted protocol data.

The important distinction is:

> **Encrypting a file does not make the file-transfer protocol secure.**

---

# Part 3 – Discover Bob's FTP Credentials

Bob's encrypted FTP credentials were stored in:

```text
bobftplogin.txt
```

The encrypted contents were copied to the CSE-LABVM and decrypted using OpenSSL:

```bash
echo '<encrypted-data>' | openssl aes-256-cbc -pbkdf2 -a -d
```

The decryption password provided by the lab was entered when prompted.

### Result

Bob's FTP credentials were successfully recovered.

The credentials were then used to access the Branch Server.

> The recovered credentials are not included in this README.

---

# Part 4 – Download Confidential Data Using FTP

From Laptop BR-2, I connected to the Branch Server:

```text
ftp 10.0.3.30
```

After authentication, I listed the available files:

```text
dir
```

The encrypted confidential file was downloaded using:

```text
get clientinfo.enc
```

The FTP session was closed with:

```text
quit
```

The file was then verified on Bob's workstation using:

```text
dir
```

### Security Observation

The downloaded file remained encrypted during the transfer.

However, because **FTP does not provide encrypted transport**, capturing the network traffic could expose FTP credentials and protocol information.

A secure alternative in real environments would be a protocol such as **SFTP** or **FTPS**, depending on the deployment requirements.

---

# Part 5 – Decrypt the Sensitive File

Bob received the decryption key through the simulated email system.

The encrypted file was copied to the CSE-LABVM and saved as:

```text
clientinfo.enc
```

The file was then decrypted using:

```bash
openssl aes-256-cbc -pbkdf2 -a -d \
-in clientinfo.enc \
-out clientinfo.txt
```

After entering the recovered decryption key, OpenSSL generated:

```text
clientinfo.txt
```

The decrypted file was then opened to verify the recovered information.

### Result

The encrypted customer-information file was successfully decrypted and its contents were recovered.

> Confidential customer information and the decryption key are intentionally not included in this repository.

---

# OpenSSL Options Used

The lab introduced several important OpenSSL options:

| Option        | Purpose                                       |
| ------------- | --------------------------------------------- |
| `aes-256-cbc` | Uses AES with a 256-bit key in CBC mode       |
| `-pbkdf2`     | Uses PBKDF2 for password-based key derivation |
| `-a`          | Handles Base64 encoding/decoding              |
| `-d`          | Performs decryption                           |
| `-in`         | Specifies the input file                      |
| `-out`        | Specifies the output file                     |

Example:

```bash
openssl aes-256-cbc -pbkdf2 -a -d -in clientinfo.enc -out clientinfo.txt
```

---

# Security Concepts Learned

## 1. Encryption Protects File Contents

The `clientinfo.enc` file could not be directly read as normal plaintext.

Encryption protects the confidentiality of the stored data when the encryption key is kept secret.

## 2. FTP Does Not Secure the Transport

FTP is an unencrypted protocol.

Even when an encrypted file is transferred, the FTP session itself can expose sensitive information.

## 3. Password-Based Encryption

The lab used:

```text
AES-256-CBC + PBKDF2
```

The password is processed through a key-derivation mechanism before being used for encryption/decryption.

## 4. Encryption Does Not Automatically Provide Integrity

The Cisco activity specifically highlights that the demonstrated AES-CBC approach does not guarantee file integrity.

An attacker could potentially modify encrypted data without the encryption scheme itself providing reliable authentication of those changes.

Modern applications should generally use an **authenticated encryption** approach, such as an appropriate AES-GCM configuration, rather than relying on CBC encryption alone.

## 5. Key Management Is Critical

Possessing an encrypted file is not enough to recover its contents.

The decryption key must also be protected.

In this lab, the key was intentionally delivered through the simulated email system so that Bob could decrypt the file.

---

# Commands Practiced

### FTP connection

```text
ftp 10.0.3.30
```

### List FTP files

```text
dir
```

### Upload a file

```text
put clientinfo.enc
```

### Download a file

```text
get clientinfo.enc
```

### Exit FTP

```text
quit
```

### Decrypt Base64-encoded data

```bash
echo '<encrypted-data>' | openssl aes-256-cbc -pbkdf2 -a -d
```

### Decrypt an encrypted file

```bash
openssl aes-256-cbc -pbkdf2 -a -d -in clientinfo.enc -out clientinfo.txt
```

### Verify files

```bash
ls
```

---

# Key Takeaways

* Encrypted files can protect the confidentiality of stored information.
* FTP does **not** provide secure transport encryption.
* File encryption and transport security are separate security controls.
* PBKDF2 strengthens password-based key derivation.
* AES-256 provides strong encryption when implemented correctly.
* CBC encryption by itself does not provide authenticated integrity.
* Encryption keys and passwords must be protected carefully.
* Modern secure systems should use authenticated encryption and secure file-transfer protocols.

---

## Lab Outcome

**Completed successfully.**

This lab provided practical experience with:

* OpenSSL
* AES-256-CBC
* PBKDF2
* Base64 encoding
* FTP authentication
* FTP file upload/download
* Encrypted file handling
* File decryption
* Confidentiality and integrity concepts
* Secure versus insecure file-transfer methods
