# Windows Endpoint Hardening & Brute Force Detection Engineering Lab

## Overview

This lab focused on hardening a Windows endpoint against brute-force authentication attacks while monitoring all security activity using Wazuh SIEM.

The environment simulated a realistic blue-team workflow involving:

- Windows endpoint hardening
- Authentication monitoring
- Brute-force attack simulation
- Detection engineering
- CIS benchmark monitoring
- Account lockout validation
- Security event analysis

The project demonstrates both:
- defensive security controls
- SOC detection capabilities

---

# Lab Environment

| System | Role |
|---|---|
| Ubuntu Server | Wazuh Manager |
| Windows 10 VM | Target Endpoint |
| Kali Linux | Attacker Machine |
| Active Directory | Centralized Authentication |

---

# Objectives

- Harden Windows authentication security
- Simulate brute-force login attacks
- Create custom Wazuh detection logic
- Monitor security events centrally
- Validate account lockout enforcement
- Analyze security telemetry inside Wazuh

---

# Technologies Used

- Wazuh SIEM
- Windows 10
- Kali Linux
- VirtualBox
- RDP / xfreerdp
- Windows Defender
- Windows Firewall
- Group Policy Editor
- MITRE ATT&CK Framework

---

# Attack Scenario

A brute-force authentication attack was simulated from Kali Linux against a Windows 10 endpoint using repeated failed RDP authentication attempts.

Wazuh monitored the endpoint and generated:
- authentication alerts
- brute-force correlation alerts
- CIS benchmark compliance alerts
- account lockout activity

---

# Step 1 — Windows Account Policy Configuration

Windows account policies were configured to improve authentication security and reduce brute-force attack effectiveness.

## Configured Policies

| Policy | Value |
|---|---|
| Account Lockout Threshold | 5 Invalid Attempts |
| Account Lockout Duration | 15 Minutes |
| Reset Lockout Counter | 15 Minutes |

---


<img src="account-policy-config.png">

---

# Step 2 — Audit Policy Configuration

Windows audit policies were configured to ensure authentication events were logged and forwarded to Wazuh.

## Enabled Policies

- Audit Logon Events
- Audit Account Logon Events
- Failure Auditing
- Success Auditing

These policies allowed:
- failed logins
- account lockouts
- authentication attempts

to appear inside Wazuh.

---

<img src="Audit policy configuraation.png">
---

# Step 3 — Windows Defender Verification

Windows Defender real-time protection was verified to ensure endpoint protection was active during testing.

## Validation

- Real-time protection enabled
- Threat protection active
- Endpoint security operational

---

<img src="Defender Active.png">
---

# Step 4 — Windows Firewall Configuration

Windows Firewall protections were reviewed and verified.

## Firewall Profiles Enabled

| Profile | Status |
|---|---|
| Domain Profile | Enabled |
| Private Profile | Enabled |

Firewall protections were maintained while allowing controlled RDP testing within the lab environment.

---

<img src="Firewall enabled.png">
---

# Step 5 — Network Level Authentication (NLA)

RDP security was hardened using Network Level Authentication (NLA).

## Security Benefit

NLA requires authentication before a remote session is fully established, reducing exposure to unauthorized access attempts.

---

<img src="NLA allowed.png">
---

# Step 6 — Password Policy Hardening

Password policies were configured to improve credential security.

## Configured Policies

| Policy | Value |
|---|---|
| Minimum Password Length | 14 Characters |
| Password Complexity | Enabled |
| Maximum Password Age | 30 Days |

---

<img src="Password policy configuration.png">
---

# Step 7 — Kali Linux Attack Configuration

Kali Linux was configured as the attacker system.

## Attack Method

Repeated failed RDP authentication attempts were generated using:

```bash
xfreerdp /u:testuser /p:WrongPassword123 /v:192.168.56.107 +auth-only
```

Multiple authentication failures were generated to trigger:
- Windows security events
- Wazuh correlation rules
- account lockout protections

---

<img src="Kali config.png">
---

# Step 8 — Custom Wazuh Detection Rule

A custom Wazuh correlation rule was created to detect repeated failed login attempts.

## Rule Configuration

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

# Detection Logic

The rule:
- monitored Windows failed login events
- triggered after 5 failures
- correlated events within 60 seconds
- escalated severity to level 10

---

# Step 9 — Wazuh Alert Detection

Wazuh successfully detected:
- failed login attempts
- authentication failures
- brute-force behavior
- CIS hardening compliance changes

## Observed Rule IDs

| Rule ID | Purpose |
|---|---|
| 60122 | Failed Windows login |
| 60104 | Windows audit failure |
| 100001 | Custom brute-force detection |
| 19009 | CIS benchmark monitoring |
| 19013 | Password policy monitoring |

---

<img src="Policy detected.png">
---

# Step 10 — Account Lockout Validation

Repeated failed authentication attempts eventually triggered Windows account lockout protections.

The lockout impacted the shared domain account across the Active Directory environment, demonstrating centralized authentication enforcement.

## Security Impact

This validated that:
- account lockout policies were functioning
- brute-force attempts were mitigated
- Windows authentication protections were active

---

<img src="User locked out after logged in attempts.png">
---

# MITRE ATT&CK Mapping

| Technique | Description |
|---|---|
| T1110 | Brute Force |
| T1078 | Valid Accounts |
| T1531 | Account Access Removal |

---

# Key Learnings

- Wazuh can correlate repeated authentication failures into high-confidence brute-force alerts.
- Windows audit policies are essential for SIEM visibility.
- Account lockout policies help mitigate password guessing attacks.
- Network Level Authentication reduces RDP exposure.
- Wazuh SCA can monitor endpoint hardening compliance.
- Active Directory authentication can impact multiple systems simultaneously.

---

# Challenges Encountered

## Domain Policy Restrictions

Some local security policies were restricted due to Active Directory domain inheritance.

### Resolution

Security settings were verified using:
- command-line policy validation
- Wazuh SCA alerts
- Windows security event analysis

---

## Field Mapping Issue

An attempt to map:

```xml
<field name="win.eventdata.ipAddress">
```

caused rule detection inconsistencies.

### Resolution

The rule was simplified to use:

```xml
<if_matched_sid>60122</if_matched_sid>
```

which restored successful correlation behavior.

---

# Future Improvements

- Configure Wazuh Active Response
- Automatically block malicious IP addresses
- Integrate Suricata IDS
- Add Sysmon telemetry
- Build custom SOC dashboards
- Add Sigma detection rules

---

# Conclusion

This lab demonstrated a full blue-team workflow involving:

- Windows endpoint hardening
- Brute-force attack simulation
- Detection engineering
- Security monitoring
- Account lockout enforcement
- Wazuh SIEM analysis

The project successfully combined:
- prevention
- detection
- monitoring
- and validation

within a realistic enterprise-style Active Directory lab environment.
