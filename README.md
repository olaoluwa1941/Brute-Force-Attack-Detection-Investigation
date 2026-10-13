# Brute-Force Attack Detection & Investigation

**A hands-on SOC investigation project focused on detecting, analyzing, correlating, and responding to suspicious authentication activity across Windows and Linux endpoints using Wazuh SIEM.**

##  Project Overview

This project simulates a Security Operations Center (SOC) investigation into suspected brute-force authentication activity within a controlled lab environment.

The objective is to demonstrate how a SOC analyst can use authentication telemetry from **Windows 11 and Ubuntu Server**, centralize the events in **Wazuh SIEM**, detect repeated authentication failures, investigate the originating system and targeted account, correlate failed and successful authentication events, identify indicators of compromise (IOCs), map observed behavior to the **MITRE ATT&CK framework**, and document appropriate incident-response actions.

The investigation focuses on authentication activity rather than simply generating alerts. The goal is to demonstrate the complete analyst workflow:

```text
Log Collection
      ↓
Detection
      ↓
Alert Validation
      ↓
Investigation
      ↓
Event Correlation
      ↓
IOC Analysis
      ↓
MITRE ATT&CK Mapping
      ↓
Severity Assessment
      ↓
Incident Response
      ↓
Documentation
```

The lab activity is performed only against systems owned and controlled within the test environment.

---

#  Project Objectives

The primary objectives of this project are to:

* Collect Windows authentication events.
* Collect Linux SSH authentication logs.
* Detect repeated failed authentication attempts.
* Identify potential brute-force behavior.
* Distinguish normal authentication failures from suspicious activity.
* Investigate source IP addresses.
* Identify targeted user accounts.
* Analyze Windows Event ID 4625.
* Correlate failed authentication events with successful authentication events.
* Analyze Linux SSH authentication failures.
* Identify potential indicators of compromise.
* Map activity to MITRE ATT&CK.
* Create SIEM detection logic.
* Perform basic alert triage.
* Determine incident severity.
* Document an investigation timeline.
* Recommend security controls and response actions.

---

# Technologies & Tools

| Technology                | Purpose                                          |
| ------------------------- | ------------------------------------------------ |
| Wazuh                     | SIEM, log collection, alerting and investigation |
| Windows 11                | Windows authentication telemetry                 |
| Ubuntu Server             | Linux SSH authentication telemetry               |
| Windows Event Viewer      | Windows event validation                         |
| Linux `/var/log/auth.log` | Linux authentication evidence                    |
| MITRE ATT&CK              | Adversary behavior mapping                       |
| PowerShell                | Windows administration and validation            |
| Bash                      | Linux administration and log analysis            |
| GitHub                    | Documentation and portfolio presentation         |

---

# Lab Architecture

```text
                         ┌──────────────────────────┐
                         │       SOC ANALYST        │
                         │                          │
                         │ Detection & Investigation│
                         └────────────┬─────────────┘
                                      │
                                      ▼
                         ┌──────────────────────────┐
                         │        WAZUH SIEM        │
                         │                          │
                         │ • Log Collection         │
                         │ • Detection              │
                         │ • Alerting               │
                         │ • Correlation            │
                         │ • Investigation          │
                         └────────────┬─────────────┘
                                      │
                         ┌────────────┴────────────┐
                         │                         │
                         ▼                         ▼
              ┌───────────────────┐     ┌───────────────────┐
              │     WINDOWS 11    │     │  UBUNTU SERVER    │
              │                   │     │                   │
              │ Security Events   │     │ SSH Authentication│
              │ Event ID 4625     │     │ auth.log          │
              │ Event ID 4624     │     │ Failed logins     │
              └───────────────────┘     └───────────────────┘
```

---

# Detection Scenario

The simulated scenario is:

> A monitored endpoint begins generating a high number of failed authentication events within a short period. The events originate from the same source and target an existing account. The SOC analyst must determine whether the behavior represents normal user error, password guessing, password spraying, or potential account compromise.

The primary detection scenario uses:

```text
5+ failed authentication attempts
within 60 seconds
```

This threshold is used specifically for the controlled lab and should be tuned against normal authentication behavior before being deployed in a production environment.

---

# Windows Investigation

## Windows Event ID 4625

Windows Event ID **4625** represents a failed logon attempt.

Important fields include:

```text
Event ID
Account Name
Source Network Address
Source Port
Logon Type
Authentication Package
Failure Reason
Status
Sub Status
Workstation Name
```

Example investigation data:

```text
Event ID:              4625
Target Account:        soc-test
Source IP:             192.168.56.30
Logon Type:            3
Authentication Package: NTLM
Status:                0xC000006D
Sub Status:             0xC000006A
```

The exact values depend on the authentication method used in the lab.

---

# Linux Investigation

Ubuntu authentication events are collected from:

```text
/var/log/auth.log
```

Example failed SSH authentication:

```text
Failed password for soc-test from 192.168.56.20
```

Authentication failures can be investigated using:

```bash
sudo grep "Failed password" /var/log/auth.log
```

Successful authentication can be searched using:

```bash
sudo grep "Accepted" /var/log/auth.log
```

The combination of failed and successful authentication events can provide important evidence during an investigation.

---

# Detection Logic

The detection hypothesis is:

> Multiple authentication failures originating from the same source within a short timeframe may indicate password guessing or another form of brute-force activity.

Example:

```text
Failed
Failed
Failed
Failed
Failed
   ↓
ALERT
```

A stronger correlation is:

```text
Failed
Failed
Failed
Failed
Failed
   ↓
Successful Authentication
   ↓
HIGHER RISK
```

---

# Investigation Methodology

The investigation follows a structured SOC workflow.

## 1. Alert Validation

Determine:

* Is the alert based on real authentication events?
* Is the source host known?
* Is the target account legitimate?
* Is the authentication activity expected?

## 2. Source Analysis

Identify:

```text
Source IP
Hostname
Network segment
Internal/external classification
```

## 3. Target Analysis

Identify:

```text
Target hostname
Target account
Account privilege level
Authentication service
```

## 4. Temporal Analysis

Determine:

```text
First failed attempt
Last failed attempt
Number of attempts
Time between attempts
Successful authentication
```

## 5. Correlation

Search for:

```text
4625 → 4625 → 4625 → 4625 → 4625
```

and:

```text
4625 → 4625 → 4625 → 4624
```

## 6. IOC Analysis

Extract:

* Source IP
* Target account
* Hostname
* Timestamp
* Authentication mechanism
* Event IDs
* Status codes
* Substatus codes
* Related suspicious activity

---

# Example Investigation Timeline

| Timestamp | Source IP     | Account  | Event | Result  |
| --------- | ------------- | -------- | ----- | ------- |
| 20:41:01  | 192.168.56.30 | soc-test | 4625  | Failed  |
| 20:41:07  | 192.168.56.30 | soc-test | 4625  | Failed  |
| 20:41:13  | 192.168.56.30 | soc-test | 4625  | Failed  |
| 20:41:19  | 192.168.56.30 | soc-test | 4625  | Failed  |
| 20:41:25  | 192.168.56.30 | soc-test | 4625  | Failed  |
| 20:41:32  | 192.168.56.30 | soc-test | 4624  | Success |

### Analyst Interpretation

Five failed authentication attempts were observed against the `soc-test` account from the same source within a short period.

A successful authentication subsequently occurred from the same source.

The sequence warrants further investigation because repeated failures followed by successful authentication may indicate that valid credentials were eventually obtained.

---

# IOC Analysis

| Indicator   | Example Value | Interpretation                    |
| ----------- | ------------- | --------------------------------- |
| Source IP   | 192.168.56.30 | Origin of authentication activity |
| Target Host | WIN-SOC01     | Target endpoint                   |
| Username    | soc-test      | Targeted account                  |
| Event ID    | 4625          | Failed Windows authentication     |
| Event ID    | 4624          | Successful Windows authentication |
| Status      | 0xC000006D    | Authentication failure            |
| Substatus   | 0xC000006A    | Bad password                      |
| Technique   | T1110         | Brute Force                       |

All values above are representative lab values.

---

# Brute Force vs Password Spraying

## Brute Force

A typical brute-force/password-guessing pattern may resemble:

```text
One Account
     ↓
Many Password Attempts
```

Example:

```text
soc-test → password attempt
soc-test → password attempt
soc-test → password attempt
soc-test → password attempt
```

## Password Spraying

Password spraying typically resembles:

```text
One Password
     ↓
Multiple Accounts
```

Example:

```text
alice
bob
charlie
david
emeka
```

The distinction is important because detection logic should consider both account concentration and account distribution.

---

# MITRE ATT&CK Mapping

## Primary Technique

**T1110 — Brute Force**

## Relevant Sub-Techniques

**T1110.001 — Password Guessing**

**T1110.003 — Password Spraying**

If credentials are successfully used after the authentication attack, the investigation may also consider:

**T1078 — Valid Accounts**

The exact technique mapping should be based on the evidence observed rather than automatically labeling every failed authentication as malicious.

---

# False Positive Analysis

Not every authentication failure is malicious.

Potential benign causes include:

* User entered an incorrect password.
* User recently changed their password.
* A service is using an outdated credential.
* A scheduled task contains an old password.
* A mapped resource contains stale credentials.
* An administrator is troubleshooting authentication.
* An application is repeatedly attempting authentication.
* A legitimate service is misconfigured.

Therefore, the SOC analyst should evaluate:

```text
Volume
+
Frequency
+
Source
+
Target
+
Account
+
Time
+
Authentication Type
+
Successful Login
+
Historical Baseline
```

before declaring an incident.

---

# Severity Assessment

## Low

```text
Small number of failed attempts
Normal user behavior
No suspicious source
No successful authentication
```

## Medium

```text
Repeated failures
Short timeframe
Same source
Same target account
```

## High

```text
Repeated failures
+
Successful authentication
+
Suspicious source
```

## Critical

```text
Successful authentication
+
Privileged account
+
Suspicious source
+
Evidence of post-authentication malicious activity
```

Severity should ultimately be determined according to the organization's incident-response policy.

---

# Recommended Response

If malicious activity is confirmed, recommended response actions include:

### Immediate Response

1. Validate the source of the activity.
2. Identify the affected account.
3. Determine whether authentication succeeded.
4. Determine the privilege level of the account.
5. Review related authentication events.
6. Review activity immediately following successful authentication.

### Containment

Depending on organizational procedures:

* Block or restrict the suspicious source.
* Disable or protect a compromised account.
* Reset compromised credentials.
* Revoke active sessions where appropriate.
* Apply additional authentication controls.

### Recovery

* Verify account ownership.
* Review endpoint activity.
* Check for lateral movement.
* Review privileged-account activity.
* Confirm no persistence mechanisms were established.
* Continue monitoring the affected systems.

### Prevention

* Enable MFA.
* Apply account lockout/rate limiting where appropriate.
* Monitor privileged accounts.
* Restrict unnecessary remote administration.
* Improve authentication monitoring.
* Centralize security logs.
* Tune SIEM detection thresholds.
* Establish authentication baselines.

---

# Repository Structure

```text
brute-force-detection-investigation/
│
├── README.md
│
├── detection/
│   ├── windows-bruteforce-rule.xml
│   └── linux-ssh-bruteforce-rule.xml
│
├── evidence/
│   ├── windows-4625-event.png
│   ├── wazuh-alert.png
│   ├── linux-auth-log.png
│   ├── successful-login.png
│   └── investigation-timeline.png
│
├── investigation/
│   ├── incident-report.md
│   ├── ioc-analysis.md
│   └── timeline.md
│
└── screenshots/
    ├── wazuh-dashboard.png
    ├── windows-event-viewer.png
    └── ubuntu-authentication-log.png
```

---

# Evidence to Include

The repository should contain screenshots demonstrating:

### 1. Windows Event Viewer

Show:

```text
Event ID 4625
```

with relevant authentication fields visible.

### 2. Wazuh Alert

Show:

```text
Possible brute-force authentication activity
```

### 3. Source IP

Show the source address associated with the authentication failures.

### 4. Target Account

Show the account being targeted.

### 5. Authentication Timeline

Show:

```text
4625
4625
4625
4625
4625
4624
```

### 6. Linux Evidence

Show failed SSH authentication entries from:

```text
/var/log/auth.log
```

### 7. Final Investigation

Show the final analyst conclusion.

---

# Sample SOC Finding

> **Finding:** Potential brute-force authentication activity was detected against the `soc-test` account.
>
> Five failed authentication attempts were observed from the same source address within a short timeframe. The activity exceeded the defined laboratory detection threshold.
>
> The authentication events were reviewed to determine whether the activity represented normal user behavior or suspicious credential-guessing activity.
>
> A subsequent successful authentication from the same source would significantly increase the severity of the event and would require additional investigation into account compromise and post-authentication activity.
>
> Based on the observed authentication pattern, the activity was mapped to MITRE ATT&CK **T1110 – Brute Force**, with **T1110.001 – Password Guessing** applicable where repeated password attempts against the same account are confirmed.

---

# Key Skills Demonstrated

This project demonstrates practical experience with:

* SIEM administration
* Wazuh
* Security monitoring
* Authentication monitoring
* Windows Security Event Logs
* Windows Event ID 4625
* Windows Event ID 4624
* Linux authentication logs
* SSH monitoring
* Log analysis
* Event correlation
* IOC identification
* Timeline analysis
* Detection engineering
* Alert triage
* False-positive analysis
* Incident severity classification
* MITRE ATT&CK
* Incident-response recommendations
* Security documentation

---

# SOC Analyst Takeaways

The major lesson from this investigation is that a single failed authentication event is usually weak evidence.

The analytical value comes from **correlation**.

```text
Single Failed Login
        ↓
Low Context
```

versus:

```text
Multiple Failed Logins
        ↓
Same Source
        ↓
Same Account
        ↓
Short Time Window
        ↓
Successful Authentication
        ↓
Potential Account Compromise
```

This project therefore focuses not only on detecting authentication failures, but on demonstrating the reasoning process required to determine whether the activity represents a genuine security incident.

---

# 📚 References

* Microsoft Learn — Windows Security Event ID 4625
* MITRE ATT&CK — T1110 Brute Force
* MITRE ATT&CK — T1110.001 Password Guessing
* Wazuh Documentation — Windows Event Log Collection
* Wazuh Documentation — Log Data Collection
* Wazuh Documentation — Failed Authentication Monitoring

---

# Lab Disclaimer

All authentication activity in this project is performed inside an isolated, authorized laboratory environment.

No unauthorized systems, accounts, networks, or third-party infrastructure are targeted.

The purpose of the project is defensive security monitoring, detection engineering, incident investigation, and SOC analyst skill development.
