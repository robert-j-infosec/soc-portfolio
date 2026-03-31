Windows Brute-Force Attack Investigation (Event ID 4625)
SOC Case Study — Splunk Detection and Analysis

Overview
This case study documents the detection and analysis of a simulated Windows authentication brute-force attack using Event ID 4625 (Failed Logon).
The objective was to validate:
- Windows Security log ingestion
- Splunk Universal Forwarder configuration
- Detection logic for brute-force patterns
- SOC investigation workflow
All logs were successfully ingested into Splunk and analyzed using custom detection queries.

Attack Simulation
Method: Network Logon Brute Force (LogonType 3)
Failed logons were generated using the Windows runas command:
runas /user:admin cmd


Entering an incorrect password repeatedly produced:
- Event ID: 4625
- LogonType: 3 (Network)
- Failure Reason: Unknown user name or bad password
- Source: WinEventLog:Security
This simulates an attacker attempting to authenticate remotely or through automated tooling.

Log Ingestion Pipeline
Splunk Universal Forwarder → Indexer → Search Head
Security logs were collected using the following inputs.conf configuration:
[WinEventLog://Security]
disabled = 0
index = wineventlog


<img width="356" height="327" alt="image" src="https://github.com/user-attachments/assets/84e4d84b-9767-4325-934c-fe2fd93870a4" />



Ingestion was verified using:
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" btool inputs list --debug

<img width="980" height="113" alt="image" src="https://github.com/user-attachments/assets/531a6a91-95f1-41c3-803d-8d0957eb95c6" />




Detection in Splunk
1. Broad Search for Failed Logons
index=* EventCode=4625


<img width="2560" height="1153" alt="image" src="https://github.com/user-attachments/assets/192fb3b4-39d9-4ffa-8c51-9b999d068fd1" />




2. Filter by Host
index=* EventCode=4625 host=MOTHERBOX


<img width="2560" height="1259" alt="image" src="https://github.com/user-attachments/assets/7aa4fbb1-d5f9-4c5e-b577-8273047cce9c" />



3. Filter by Targeted Account
index=* EventCode=4625 Account_Name=admin


<img width="2560" height="1256" alt="image" src="https://github.com/user-attachments/assets/8ef24141-ba65-49f7-ad9a-8fa60808af63" />




Timeline Analysis
A brute-force attack typically shows rapid, repeated failures over a short time window.
Query
index=* EventCode=4625
| bin _time span=1m
| stats count by _time


<img width="2560" height="495" alt="image" src="https://github.com/user-attachments/assets/d649c332-4941-496b-a205-784d22dcebec" />




Summary of Findings
Query
index=* EventCode=4625
| stats count values(Failure_Reason) as FailureReasons values(Logon_Type) as LogonTypes by Account_Name, host

<img width="2560" height="366" alt="image" src="https://github.com/user-attachments/assets/2b843b8d-2ec3-4d31-9992-114398cb329d" />


Key Observations
- Multiple failed logons occurred in rapid succession
- Targeted account: admin
- LogonType: 3 (Network)
- Failure Reason: Unknown user name or bad password
- Source Host: MOTHERBOX
This pattern is consistent with brute-force or password-spray activity.

Root Cause Analysis
The failed logons were intentionally generated as part of a security lab exercise.
In a real environment, this pattern would indicate:
- Credential guessing
- Unauthorized access attempts
- Automated brute-force tools
- Malware attempting lateral movement

Mitigation Recommendations
Authentication Controls
- Enforce account lockout policies
- Require strong password complexity
- Implement multi-factor authentication
Monitoring and Alerting
- Create Splunk alerts for:
- Excessive 4625 events
- Multiple failures against a single account
- Multiple failures across many accounts
Hardening
- Disable unused accounts
- Limit remote logon permissions
- Audit privileged accounts regularly

Conclusion
This investigation demonstrates:
- Successful ingestion of Windows Security logs
- Detection of brute-force authentication attempts
- Ability to analyze Event ID 4625 patterns
- SOC-level investigation workflow
- Clear, actionable mitigation strategies
This case study is fully GitHub-ready and showcases your ability to:
- Configure Splunk
- Collect Windows logs
- Detect authentication attacks
- Perform timeline and pattern analysis
- Produce a professional SOC incident report



