# SOC Investigation Lab — Windows Incident Response

A personal project where I investigated a simulated attack on a Windows machine using Splunk.
The goal was to practice real SOC analyst work, detecting, investigating, and documenting a security incident.

---

## Tools Used
- Splunk (SIEM)
- Windows Event Logs
- Sysmon Logs

---

## What Happened (Short Version)

An attacker from 10.0.0.47 targeted a Windows server at 10.0.0.20.
They scanned the network, brute forced their way in, uploaded a web shell,
got a reverse shell, escalated to SYSTEM, and set up persistence before getting caught.

---

## Project Files

- logs/ — Raw log file used for investigation
- investigation/ — Full investigation report with timeline and findings
- queries/ — All Splunk queries I used during the investigation
- screenshots/ — Screenshots taken directly from Splunk

---

## What I Practiced

- Detecting brute force attacks in Splunk
- Identifying web shell and reverse shell activity
- Tracing privilege escalation in Windows logs
- Mapping findings to MITRE ATT&CK
- Writing an incident report

---

## MITRE ATT&CK Coverage

T1046 T1110 T1078 T1505.003 T1059.001 T1068 T1136.001 T1547.001 T1053.005 T1021.001
