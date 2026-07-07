# Investigation Report — Windows Security Incident

**Date:** 2024-03-15
**Analyst:** Menard Francisco
**Severity:** Critical
**Status:** Resolved

---

## Summary

A Windows server (10.0.0.20) was compromised by an attacker from IP 10.0.0.47.
The attacker performed a port scan, brute forced their way in as svc_web,
executed a web shell, established a reverse shell, escalated to SYSTEM,
and set up multiple persistence mechanisms before being detected.

---

## Affected Systems

Host: CORP-SRV01
IP: 10.0.0.20
OS: Windows Server

---

## Attacker Information

IP: 10.0.0.47
Workstation Name: KALI-PC
First Seen: 2024-03-15 02:13:01 UTC
Last Seen: 2024-03-15 02:20:01 UTC

---

## Timeline of Events

02:13 UTC — Attacker scanned victim machine on ports 80, 443, 445, 3389, 8080, 135, 5985

02:15 UTC — Brute force attack started against admin, administrator, and svc_web accounts

02:15 UTC — Brute force succeeded. Attacker logged in as svc_web using NTLM authentication

02:16 UTC — Attacker executed commands through a web shell located in C:\inetpub\wwwroot\upload\cmd.aspx

02:16 UTC — Attacker ran whoami, net user, net localgroup administrators to enumerate the system

02:17 UTC — Attacker ran encoded PowerShell command that downloaded and executed a reverse shell

02:17 UTC — Reverse shell connection established from victim to attacker on port 4444

02:18 UTC — Attacker ran PrintSpoofer64.exe to abuse SeImpersonatePrivilege and escalate to SYSTEM

02:19 UTC — Attacker created backdoor user account named hacker and added to Administrators group

02:19 UTC — Attacker added beacon.exe to registry Run key as WindowsUpdate for persistence

02:20 UTC — Attacker created scheduled task named Microsoft\Windows\Update to run beacon.exe on startup

---

## Findings

### 1. Port Scan

Attacker scanned 7 ports in under 2 seconds.
This is not normal user behavior and indicates automated scanning.

### 2. Brute Force

9 failed login attempts across 3 usernames before succeeding.
No account lockout policy was in place which allowed this to continue.

### 3. Initial Access

svc_web account was compromised.
The account had SeImpersonatePrivilege which gave the attacker
an easy path to SYSTEM.

### 4. Web Shell

A file named cmd.aspx was found in the web upload directory.
This was used to run commands on the server remotely.
This means the attacker also had access to the web application
before the brute force or used it as another entry point.

### 5. Reverse Shell

PowerShell was used with -nop -w hidden -enc flags to hide the command.
The encoded payload downloaded rev.ps1 from the attacker machine
and executed it which called back to port 4444.

### 6. Privilege Escalation

PrintSpoofer64.exe was uploaded to C:\Windows\Temp and executed.
This tool abuses SeImpersonatePrivilege to gain SYSTEM access.
Confirmed by user changing from svc_web to NT AUTHORITY\SYSTEM in logs.

### 7. Persistence

Four persistence methods were found:

First — Backdoor user account named hacker was created and added to Administrators.

Second — beacon.exe added to HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
disguised as WindowsUpdate.

Third — Scheduled task created named Microsoft\Windows\Update
set to run beacon.exe on every system startup as SYSTEM.

Fourth — RDP was enabled by setting fDenyTSConnections to 0
allowing the attacker to come back using the hacker account via RDP.

---

## MITRE ATT&CK Mapping

T1046 — Network Service Discovery (Port Scan)
T1110 — Brute Force
T1078 — Valid Accounts (svc_web)
T1505.003 — Web Shell (cmd.aspx)
T1059.001 — PowerShell
T1071.001 — Web Protocols (C2 on port 4444)
T1068 — Exploitation for Privilege Escalation (PrintSpoofer)
T1136.001 — Create Local Account (hacker)
T1547.001 — Registry Run Keys (beacon.exe)
T1053.005 — Scheduled Task (beacon.exe)
T1021.001 — Remote Desktop Protocol

---

## Recommendations

1. Block IP 10.0.0.47 at the firewall immediately

2. Disable and investigate the svc_web account

3. Delete the hacker user account

4. Remove beacon.exe from the registry Run key

5. Delete the scheduled task named Microsoft\Windows\Update

6. Remove cmd.aspx from C:\inetpub\wwwroot\upload\

7. Implement account lockout policy
   Suggested: Lock account after 5 failed attempts

8. Review which accounts have SeImpersonatePrivilege
   and remove it if not needed

9. Monitor C:\Windows\Temp for executables
   This is a common drop location for attackers

10. Enable PowerShell Script Block Logging
    to catch encoded commands in the future
