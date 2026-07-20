Detection & Incident Report Portfolio
**Environment:** 3-VM Home Lab (Kali Linux Attacker, Windows 11 Victim, Wazuh-Manager2 SIEM)  
**Date:** July 20, 2026  

---

##  Introduction
This document consolidates three detection findings generated while validating a self-hosted Wazuh SIEM deployment against simulated attacker activity in an isolated home lab. Each finding follows a SOC-analyst-style structure: activity executed, detection status, underlying log source/rule, and analyst next steps.

---

## Report 1: Failed Authentication Attempts (Suspected Credential Brute-Force)
* **Report ID:** INC-001
* **Date/Time:** July 20, 2026, 18:18:22 - 18:30:58
* **Severity:** Low (Lab test activity)
* 
###  Summary
Five failed NTLM authentication attempts were detected against host `Victim-Windows` (`192.168.100.20`) originating from `Kali` (`192.168.100.10`). The attempts used non-existent usernames, consistent with account enumeration.

###  Detection Details
* **Detection Source:** Wazuh SIEM (`Wazuh-Manager2`)
* **Underlying Event:** Windows Security Event ID 4625 (*"An account failed to log on"*)
* **Wazuh Rule Fired:** `rule.id 60122` - *"Logon Failure - Unknown user or bad password"* (Level 5)
* **Volume:** 5 events in a ~12-minute window

###  Log Analysis Highlights
* `authenticationPackageName`: **NTLM** (Expected for SMB auth)
* `targetUserSid`: **S-1-0-0** (Null SID, confirming non-existent account)
* `targetDomainName`: **WORKGROUP** (Local SAM auth)

###  Root Cause
Test activity using `smbclient -L` from Kali against the Windows target's SMB share with invalid credentials to verify Wazuh log ingestion.

###  Analyst Assessment & Response
In production, multiple failed logons across usernames indicate account enumeration. Recommended actions include blocking the source IP, isolating the target host, and checking for follow-up successful logins.

---

## Report 2: EICAR Test File Detection & SIEM Visibility Remediation
* **Report ID:** INC-002
* **Date/Time:** July 20, 2026, ~20:42 UTC
* **Severity:** Low (Controlled AV test file)

### Summary
Microsoft Defender Antivirus detected and quarantined an EICAR test file downloaded to `Victim-Windows` via PowerShell (`Invoke-WebRequest`). An initial telemetry gap was identified and remediated during testing.

### Detection Details
* **Underlying Event:** Windows Event ID 1117 (*"Microsoft Defender Antivirus has detected malware"*)
* **Event Channel:** `Microsoft-Windows-Windows Defender/Operational`
* **Wazuh Rule Fired:** `rule.id 62124` - Windows Defender detection (Level 3)
* **Threat Identified:** `Virus:DOS/EICAR_Test_File`
* **File Path:** `C:\Users\ars\Desktop\eicar_test2.txt`

###  Telemetry Gap & Remediation
The default Wazuh agent `ossec.conf` did not include the Windows Defender operational channel. 

**Fix:** Added the following configuration block to `ossec.conf` and restarted the Wazuh agent service:

xml
<localfile>
  <location>Microsoft-Windows-Windows Defender/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
End-to-end detection from Windows Event ID 1117 to Wazuh Rule 62124 was confirmed upon re-testing.

 Report 3: Detection Gap Analysis – Network Port Scanning
Report ID: FIND-003

Date/Time: July 20, 2026

Type: Detection Capability Gap

Summary
An nmap -sV -A scan was executed from Kali (192.168.100.10) against Victim-Windows (192.168.100.20), successfully identifying open ports (135/msrpc, 139/netbios-ssn, 445/microsoft-ds). However, no alerts were generated in Wazuh.

 Root Cause Analysis
Agent Query Exclusion: The default ossec.conf excludes Windows Filtering Platform events (Event IDs 5145 and 5156).

Audit Policy Limitations: Windows advanced network object auditing is disabled by default.

Host-Based Boundary: Raw TCP SYN handshakes do not generate host authentication or process execution events.

 Recommendations to Close Gap
Deploy Sysmon on endpoints for granular network connection tracking (Event ID 3).

Integrate a network-based IDS (Suricata or Snort) to inspect raw network traffic alongside host logs.

Conclusion
This lab validated Wazuh's ability to detect brute-force and malware events out of the box while surfacing a real, fixable telemetry gap (Defender log forwarding). Identifying host-based logging limitations (Nmap port scanning) highlights the necessity of layering endpoint visibility with network IDS tooling in production SOC environments.
