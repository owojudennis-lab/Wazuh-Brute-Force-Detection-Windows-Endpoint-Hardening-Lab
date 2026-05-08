# Wazuh Brute Force Detection & Windows Endpoint Hardening Lab

## Overview

This lab demonstrates how to:

- Deploy and configure Wazuh for Windows security monitoring
- Detect brute force login attempts using custom Wazuh rules
- Simulate failed authentication attacks from Kali Linux
- Configure Windows endpoint hardening policies
- Validate security alerts and audit events inside Wazuh
- Map detections to MITRE ATT&CK techniques

---

# Lab Architecture

| System | Role | IP Address |
|---|---|---|
| Wazuh Server | SIEM / Detection Platform | 192.168.56.105 |
| Windows 10 VM | Target Endpoint | 192.168.56.107 |
| Kali Linux | Attacker Machine | Host-Only Network |

---

# Objectives

- Configure Windows event monitoring with Wazuh
- Generate failed authentication events
- Create a custom brute-force detection rule
- Simulate attack activity from Kali Linux
- Harden Windows security policies
- Analyze alerts inside the Wazuh dashboard

---

# Tools & Technologies

- Wazuh
- Windows 10
- Kali Linux
- VirtualBox
- RDP / xfreerdp
- Windows Group Policy Editor
- MITRE ATT&CK Framework

---

# MITRE ATT&CK Mapping

| Technique | Description |
|---|---|
| T1110 | Brute Force |
| T1078 | Valid Accounts |
| T1531 | Account Access Removal |
| T1070 | Indicator Removal (potential defensive logging scenarios) |

---

# Step 1 — Verify Wazuh Agent Connectivity

Verified that the Windows endpoint was successfully connected to the Wazuh manager.

## Validation

- Wazuh dashboard showed the Windows agent as active
- Security events were being collected successfully

---

---
<img src="Policy detected.png">
---

# Step 2 — Generate Failed Login Events

Failed login attempts were generated against the Windows 10 endpoint to create authentication failure logs.

## Attack Simulation

From Kali Linux:

```bash
xfreerdp /u:Administrator /p:wrongpass /v:192.168.56.107 +auth-only
```

Multiple failed attempts were generated using:

```bash
for i in {1..10}; do xfreerdp /u:Administrator /p:wrongpass /v:192.168.56.107 +auth-only; done
```

---

## Screenshot Placeholders

### Screenshot 2
**Kali Linux terminal showing brute-force simulation command**

### Screenshot 3
**Failed authentication output from xfreerdp**

---

# Step 3 — Observe Native Wazuh Authentication Alerts

Wazuh successfully detected failed Windows logon attempts.

## Observed Alerts

| Rule ID | Description |
|---|---|
| 60122 | Logon failure — Unknown user or bad password |
| 60104 | Windows audit failure event |

These alerts confirmed that Windows authentication events were reaching the SIEM correctly.

---

## Screenshot Placeholder

### Screenshot 4
**Wazuh security events showing failed login alerts**

---

# Step 4 — Create Custom Brute Force Detection Rule

A custom Wazuh rule was created to detect repeated failed login attempts.

## Custom Rule

File:

```bash
/var/ossec/etc/rules/local_rules.xml
```

Rule:

```xml
<group name="custom_rules,authentication">

  <rule id="100001" level="10" frequency="5" timeframe="60">
    <if_matched_sid>60122</if_matched_sid>
    <description>Multiple failed login attempts detected (possible brute force)</description>
    <group>authentication_failed,pci_dss_10.2.4,pci_dss_10.2.5</group>
  </rule>

</group>
```

---

## Rule Logic

- Trigger after 5 failed login attempts
- Timeframe: 60 seconds
- Based on parent rule SID 60122
- Severity level increased to 10

---

## Screenshot Placeholder

### Screenshot 5
**Custom rule inside local_rules.xml**

---

# Step 5 — Restart Wazuh Manager

After adding the custom rule, the Wazuh manager was restarted.

## Commands

```bash
sudo systemctl restart wazuh-manager
```

Validation:

```bash
sudo systemctl status wazuh-manager
```

---

## Screenshot Placeholder

### Screenshot 6
**Successful Wazuh manager restart**

---

# Step 6 — Trigger Custom Brute Force Detection

The brute-force simulation was executed again after enabling the custom rule.

Wazuh successfully correlated repeated failed logins into a higher-severity brute-force alert.

## Generated Alert

| Rule ID | Description |
|---|---|
| 100001 | Multiple failed login attempts detected (possible brute force) |

---

## Screenshot Placeholder

### Screenshot 7
**Custom brute-force detection alert inside Wazuh**

---

# Step 7 — Windows Endpoint Hardening

Windows security policies were hardened using Local Group Policy Editor.

## Security Policies Configured

### Password Policy

Path:

```text
Computer Configuration
→ Windows Settings
→ Security Settings
→ Account Policies
→ Password Policy
```

Configured:

| Setting | Value |
|---|---|
| Minimum Password Length | 14 |
| Password Complexity | Enabled |
| Maximum Password Age | 30 Days |

---

### Account Lockout Policy

Path:

```text
Account Policies
→ Account Lockout Policy
```

Configured:

| Setting | Value |
|---|---|
| Account Lockout Threshold | 5 invalid attempts |
| Lockout Duration | 15 minutes |
| Reset Counter After | 15 minutes |

---

## Screenshot Placeholders

### Screenshot 8
**Password policy configuration**

### Screenshot 9
**Account lockout policy configuration**

---

# Step 8 — Validate Hardening Events in Wazuh

Wazuh generated CIS benchmark and hardening-related alerts after policy modifications.

## Observed Alerts

| Rule ID | Description |
|---|---|
| 19013 | Minimum password length configured |
| 19015 | Guest account disabled |
| 19009 | Anonymous SID translation disabled |

These alerts confirmed that Wazuh Security Configuration Assessment (SCA) was functioning properly.

---

## Screenshot Placeholder

### Screenshot 10
**Wazuh CIS benchmark compliance alerts**

---

# Findings

- Wazuh successfully detected failed authentication attempts
- Custom correlation rules improved brute-force visibility
- Windows hardening policies generated compliance events
- MITRE ATT&CK mappings were automatically applied
- Endpoint telemetry was successfully centralized into the SIEM

---

# Challenges Encountered

## Rule Mapping Issue

Attempted IP-based field mapping using:

```xml
<field name="win.eventdata.ipAddress">
```

The rule initially failed due to field mapping inconsistencies.

### Resolution

The rule was simplified to use only:

```xml
<if_matched_sid>60122</if_matched_sid>
```

This restored successful alert generation.

---

# Key Lessons Learned

- Correlation rules improve SIEM detection quality
- Windows Event IDs are critical for authentication monitoring
- Wazuh custom rules require careful field mapping validation
- Endpoint hardening can be validated through Wazuh SCA alerts
- Brute-force detection can be simulated safely in a lab environment

---

# Future Improvements

- Integrate Suricata IDS with Wazuh
- Add active response for automatic IP blocking
- Forward logs into a dedicated SOC dashboard
- Configure email alerting
- Add Sysmon telemetry
- Simulate lateral movement scenarios

---

# Conclusion

This lab demonstrated a complete workflow for:

- Windows security monitoring
- Brute-force attack detection
- SIEM rule customization
- Endpoint hardening
- Security event analysis using Wazuh

The environment successfully simulated real-world SOC monitoring and defensive detection engineering workflows.

---
