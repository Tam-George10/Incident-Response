###  User Home Directory Enumeration – Hidden Files

#### Objective
Identify suspicious or hidden files within the `giorgio` user’s home directory that may indicate persistence mechanisms or attacker artifacts.

---

#### Method Used

```bash
ls -la

```
The ls -la command was executed immediately after logging into the account to list all files, including hidden files (those prefixed with a dot), along with their permissions and ownership.

## Findings
Suspicious File Identified

Filename: .bad_bash

Location: /home/giorgio/

## Analysis
Hidden files are commonly abused by attackers to conceal malicious scripts or configuration files. The .bad_bash file stood out due to:

Its non-standard naming

Its hidden nature

Lack of a legitimate purpose in a default Linux user environment

This discovery indicated possible user-level persistence or tampering and warranted further inspection of shell configuration files such as .bashrc.

## Why This Matters
Enumerating hidden files is a fundamental step in Linux incident response. It allows responders to:

Detect attacker artifacts early

Identify persistence mechanisms

Establish whether a user account has been modified

This step provided the first concrete indicator that the giorgio account had been compromised.
