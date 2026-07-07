# Splunk Queries — Incident Investigation

## 1. View All Logs

index=main
| sort \_time

Just to see everything in order before I start investigating.

---

## 2. Port Scan Detection

index=main EventCode=3 Image=nmap.exe
| table \_time SrcIP DstIP DstPort Protocol
| sort \_time

Noticed multiple ports being hit within seconds from the same IP.
Not normal behavior.

---

## 3. Failed Login Attempts

index=main EventCode=4625
| table \_time TargetUserName IpAddress FailureReason
| sort \_time

Multiple failed logins from 10.0.0.47 across different usernames.

---

## 4. Count Failed Logins Per IP

index=main EventCode=4625
| stats count by IpAddress
| sort -count

10.0.0.47 had the most failed attempts. Confirms brute force.

---

## 5. Successful Login After Failed Attempts

index=main EventCode=4624 OR EventCode=4625
| table \_time EventCode TargetUserName IpAddress
| sort \_time

Same IP eventually got in as svc_web after multiple failures.

---

## 6. Check Account Privileges

index=main EventCode=4672
| table \_time SubjectUserName PrivilegeList

svc_web had SeImpersonatePrivilege which is dangerous
and can be used for privilege escalation.

---

## 7. Web Shell Activity

index=main EventCode=1 ParentImage=_cmd.aspx_
| table \_time User ParentImage Image CommandLine

Commands being executed through a web shell in the upload folder.
Attacker was running basic recon commands after getting in.

---

## 8. Suspicious PowerShell

index=main EventCode=1 Image=_powershell_
| table \_time User CommandLine

PowerShell ran with -nop -w hidden -enc flags.
Encoded command that downloaded and ran a reverse shell.

---

## 9. Outbound Connection on Port 4444

index=main EventCode=3 DstPort=4444
| table \_time User Image SrcIP DstIP DstPort

Victim machine called back to attacker on port 4444.
Classic reverse shell indicator.

---

## 10. Privilege Escalation Tool

index=main EventCode=1 Image=_PrintSpoofer_
| table \_time User ParentImage CommandLine

PrintSpoofer64.exe ran from Temp folder.
User went from svc_web to SYSTEM after this.

---

## 11. New User Account Created

index=main EventCode=4720 OR EventCode=4732
| table \_time EventCode TargetUserName SubjectUserName

User named hacker was created and added to Administrators group
by SYSTEM at 2AM. Clear backdoor account.

---

## 12. Registry Persistence

index=main EventCode=13
| table \_time User TargetObject Details

beacon.exe added to the Run registry key.
Named it WindowsUpdate to blend in.

---

## 13. Scheduled Task Persistence

index=main EventCode=4698
| table \_time TaskName TaskContent SubjectUserName

Scheduled task created called Microsoft\Windows\Update.
Also runs beacon.exe on startup. Second persistence method.

---

## 14. Full Attack Timeline

index=main
| table \_time EventCode User Image CommandLine SrcIP DstIP DstPort
| sort \_time

Full picture of the attack from start to finish.

---

## 15. Event Summary

index=main
| stats count by EventCode
| sort -count

Quick overview of how many of each event type occurred.
