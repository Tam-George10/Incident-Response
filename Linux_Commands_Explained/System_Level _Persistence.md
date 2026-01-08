###  System-Level Persistence – Abused Default Account (nobody)

#### Objective
Identify remaining persistence mechanisms by inspecting default system accounts that may be abused by attackers to maintain access while blending into normal system behavior.

---

#### Method Used

```bash
cat /etc/passwd | grep -i "nobody"

```
This command was used to search the system’s user account database for entries related to the default nobody account.

## Observations
The nobody account was present in the system, consistent with a standard Linux installation. However, during earlier investigation steps, attacker-controlled behavior was observed to be associated with this account, indicating it had been abused as part of the persistence strategy.

Default accounts such as "nobody" are often overlooked because they:

Exist on fresh installations

Are expected to have minimal privileges

Typically do not require regular user interaction

## Analysis
The attacker leveraged the "nobody" account to conceal persistence within what appears to be a legitimate system component. By associating malicious activity with an existing low-privilege account, the attacker reduced the likelihood of detection during casual account reviews.

This technique relies on the assumption that defenders will focus primarily on custom or recently created users, allowing modified default accounts to persist unnoticed.

## Why This Matters
System accounts like "nobody" should be treated with the same scrutiny as standard user accounts during incident response. Attackers frequently abuse:

Default users

Service accounts

Low-privilege system identities

to maintain stealthy access. Identifying misuse of these accounts is often the final step in uncovering deeply embedded persistence mechanisms.

This finding completed the identification of all known backdoors on the compromised system and allowed the incident response team to proceed with full remediation and recovery.
