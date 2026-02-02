## 🧩 What “Vulnerable Templates” Means in Certipy‑AD

Certipy‑AD analyzes **certificate templates** published in AD CS and flags those whose configuration allows unintended privilege escalation. These templates define _who_ can request certificates and _what_ those certificates can be used for.

A template becomes “vulnerable” when its settings allow a requester to obtain a certificate that AD will treat as a **logon credential** for another identity.

Certipy‑AD automates the discovery of these misconfigurations.

## 🔥 The Core Vulnerabilities Certipy Looks For

Below is a structured breakdown of the template misconfigurations Certipy‑AD highlights.

### 1. **ESC1 – Enrollment Rights + Client Authentication**

A template is vulnerable if:

- A low‑privileged user/group can **enroll**, and
    
- The template allows **Client Authentication** or **Smartcard Logon**, and
    
- The subject name can be **chosen by the requester**.
    

This allows impersonation of _any_ domain user or computer.

### 2. **ESC2 – Certificate Request Agent (Enrollment Agent)**

If a template allows a user to enroll as a **Certificate Request Agent**, they can request certificates _on behalf of others_.

If enrollment rights are too broad → privilege escalation.

### 3. **ESC3 – Dangerous EKUs + Misconfigured Issuance Requirements**

Templates that:

- Allow enrollment
    
- Include **Any Purpose** EKU or **No EKU**
    
- Lack proper issuance requirements
    

…can be used to mint certificates that AD treats as authentication tokens.

### 4. **ESC4 – Vulnerable CA ACLs**

Not a template issue per se, but Certipy reports it alongside templates.

If a user can:

- Manage CA permissions
    
- Modify templates
    
- Issue certificates
    

…they can escalate to domain admin.

### 5. **ESC5 – Misconfigured Certificate Authorities**

If the CA trusts templates that allow dangerous EKUs or subject name control, Certipy flags them.

### 6. **ESC6 – NTAuth Store Misconfigurations**

If a template is trusted for authentication but the CA is not properly constrained, attackers can exploit it.

### 7. **ESC7 – Vulnerable Subordinate CAs**

If a subordinate CA is misconfigured, an attacker can issue arbitrary certificates.

## 🧭 What Certipy‑AD Actually Outputs

When you run:

```
certipy-ad find -u <user> -p <pass> -dc-ip <ip> -vulnerable
```

You get a list of templates with:

- Template name
    
- Enrollment permissions
    
- EKUs
    
- Subject name requirements
    
- Whether the template is exploitable (ESC1–ESC7)
    

It’s essentially a **PKI attack surface map**.

## 🛠 Why This Matters Architecturally

A vulnerable template is equivalent to:

**“A domain‑trusted identity minting mechanism that accepts untrusted input.”**

In AD CS, that’s catastrophic because certificates can be used to:

- Authenticate over Kerberos (PKINIT)
    
- Authenticate over LDAP/LDAPS
    
- Request TGTs
    
- Persist access
    

This is why Certipy‑AD’s template analysis is so central to modern AD security reviews.

### Certipy version

You have to install, if you don't have it yet, the certipy v.5.0.3 or superior.

```
sudo apt update
sudo apt install certipy-ad
```

---
## Running

#Attacking_machine 

```bash
certipy-ad find -u 'gMSA_CA_prod$' -hashes '02dc1e938b2004f0c8145b499ee73f41' -dc-ip <MACHIINE IP> -vulnerable
```

![[certipy.png]]

- Look at the *20260123101632_Certipy.txt* or similar.

![[cert_text_report.png]]

Great! You've identified **ESC7**, which is a serious Active Directory Certificate Services (AD CS) vulnerability. This means the user has dangerous permissions that can be exploited for privilege escalation.

Now we can pivot from `ESC7` to `ESC6`.

**Next step:** [[Configuration for ESC7 to ESC6]]