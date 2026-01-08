# 01 – OS Identification (Initial Access)

## Objective
Identify the operating system of the compromised server immediately after access. Establishing the OS baseline helps distinguish default system behavior from attacker-added persistence mechanisms later in the investigation.

---

## Method Used

```bash
ssh giorgio@<TARGET_IP>

```
## Explanation:
Upon successful SSH login, the system automatically displayed operating system and kernel information as part of the login banner. No additional enumeration commands were required at this stage.

Ubuntu systems commonly present this information via:

/etc/update-motd.d/

/etc/os-release

PAM login banners

## Findings

Operating System: Ubuntu 20.04.6 LTS

Kernel: Linux (x86_64)

Access Level: User account with elevated (root-capable) privileges

## Incident Response Relevance

Confirms the system is a long-term support (LTS) release, commonly targeted due to widespread use

Helps narrow down default persistence locations such as:

/etc/cron*

/etc/systemd/

User and root .bashrc / .profile files

Establishes expected baseline users (e.g., root, nobody)

## Analyst Insight

OS fingerprinting is a mandatory first step in incident response. Without understanding what is normal for the system, it is impossible to accurately identify malicious persistence mechanisms or attacker activity.
