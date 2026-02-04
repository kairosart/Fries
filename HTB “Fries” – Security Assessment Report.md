
**Prepared for:** Internal Security Review  
**Prepared by:** Kairos  
**Classification:** Confidential  
**Date:** _02-04-2026_

---

## 1. Executive Summary

The Fries environment simulates a modern enterprise network composed of a public‑facing Ubuntu web server, Dockerized application components, an NFS share, and a Windows Server 2022 Active Directory domain with AD CS. The assessment identified multiple weaknesses across credential management, container isolation, file‑share configuration, and certificate services. When chained, these issues enable full domain compromise.

**Overall Risk Rating:** Critical

---

## 2. Scope of Assessment

### In‑Scope Components

- Public web application (Ubuntu host)
    
- Docker containers and backend services
    
- NFS share
    
- Windows Server 2022 Domain Controller
    
- Active Directory + AD CS
    
- Internal management interfaces (pgAdmin, LDAP, etc.)
    

### Testing Approach

- Passive and active enumeration
    
- Configuration review
    
- Credential and access control analysis
    
- AD CS template and enrollment review
    
- Network segmentation evaluation
    

---

## 3. High‑Level Attack Path

1. Weak pgAdmin credentials allowed unauthorized access to database management.
    
2. Docker misconfigurations enabled container‑to‑host interaction.
    
3. Insecure NFS share exposed sensitive configuration files.
    
4. LDAP credential exposure enabled lateral movement into AD.
    
5. AD CS misconfigurations (ESC6 / ESC16) allowed privilege escalation to Domain Admin.
    

This chain demonstrates how medium‑severity issues can combine into a critical compromise.

---

## 4. Findings & Vulnerabilities

### 4.1 Weak or Exposed Application Credentials

**Severity:** High  
**Description:** Administrative interfaces (e.g., pgAdmin) used weak or reused credentials. No MFA or IP restrictions were applied.  
**Impact:** Unauthorized access to database management and sensitive configuration data.  
**Remediation:**

- Enforce strong, unique passwords.
    
- Implement MFA for all administrative interfaces.
    
- Restrict access via VPN or IP allowlists.
    
- Rotate credentials and monitor for reuse.
    

### 4.2 Docker Misconfiguration Allowing Host Interaction

**Severity:** High  
**Description:** Containers lacked proper isolation and were able to interact with host resources.  
**Impact:** Compromise of a container could lead to compromise of the host.  
**Remediation:**

- Disable privileged containers.
    
- Remove unnecessary host mounts.
    
- Apply AppArmor/SELinux profiles.
    
- Use Docker rootless mode where possible.
    
- Implement runtime monitoring (Falco, Auditd).
    

### 4.3 Insecure NFS Share Configuration

**Severity:** High  
**Description:** NFS exports allowed unintended read access to sensitive files.  
**Impact:** Exposure of credentials and configuration data enabling lateral movement.  
**Remediation:**

- Restrict exports to specific hosts.
    
- Enable `root_squash`.
    
- Remove sensitive data from shared directories.
    
- Use NFSv4 with Kerberos authentication.
    

### 4.4 LDAP Credential Leakage / Anonymous Bind Issues

**Severity:** High  
**Description:** LDAP allowed excessive information disclosure or credential exposure.  
**Impact:** Attackers can authenticate or escalate privileges within AD.  
**Remediation:**

- Disable anonymous binds.
    
- Enforce LDAPS.
    
- Harden password policies.
    
- Limit attribute exposure.
    

### 4.5 AD CS Misconfigurations (ESC6 / ESC16)

**Severity:** Critical  
**Description:** Certificate templates allowed dangerous enrollment configurations, including user‑supplied subject names and overly permissive enrollment rights.  
**Impact:** Attackers can forge authentication certificates and escalate to Domain Admin.  
**Remediation:**

- Audit all certificate templates.
    
- Remove “ENROLLEE_SUPPLIES_SUBJECT” where not required.
    
- Restrict enrollment permissions.
    
- Require manager approval for sensitive templates.
    
- Follow Microsoft AD CS hardening guidelines.
    

### 4.6 Lack of Network Segmentation

**Severity:** Medium  
**Description:** Web, application, NFS, and AD services were reachable without segmentation.  
**Impact:** Compromise of one service leads to rapid lateral movement.  
**Remediation:**

- Segment networks by role.
    
- Apply strict firewall rules.
    
- Use jump hosts for administrative access.
    

### 4.7 Insufficient Monitoring & Logging

**Severity:** Medium  
**Description:** No centralized logging or alerting was in place.  
**Impact:** Malicious activity may go undetected.  
**Remediation:**

- Deploy SIEM (Wazuh, Splunk, ELK).
    
- Enable AD auditing (logon events, certificate enrollment).
    
- Monitor Docker and NFS activity.
    

---

## 5. Risk Rating Summary

|Finding|Severity|Impact|
|---|---|---|
|Weak Credentials|High|Unauthorized access|
|Docker Misconfigurations|High|Host compromise|
|NFS Misconfiguration|High|Credential leakage|
|LDAP Issues|High|AD compromise|
|AD CS Misconfigurations|Critical|Domain Admin escalation|
|Lack of Segmentation|Medium|Lateral movement|
|Insufficient Logging|Medium|Undetected compromise|

---

## 6. Remediation Roadmap

### Immediate (0–7 Days)

- Rotate all credentials.
    
- Restrict access to administrative interfaces.
    
- Disable anonymous LDAP binds.
    
- Harden Docker containers.
    

### Short Term (1–4 Weeks)

- Fix NFS permissions.
    
- Implement segmentation and firewall rules.
    
- Harden AD CS templates.
    

### Medium Term (1–3 Months)

- Deploy centralized logging.
    
- Implement MFA across all administrative accounts.
    
- Conduct internal security training.
    

---

## 7. Conclusion

The Fries environment contains multiple misconfigurations that, when chained, enable full domain compromise. Addressing these issues requires improvements in credential hygiene, container security, AD CS configuration, and network segmentation. Implementing the recommended remediations will significantly reduce the attack surface and strengthen the overall security posture.