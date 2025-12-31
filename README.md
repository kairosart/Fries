# Fries
The HackTheBox machine "Fries" is a hard difficulty challenge that involves a multi-stage attack on a complex Active Directory infrastructure with interconnected systems, including a Windows Server 2022 Domain Controller and an Ubuntu web server. 

## Nmap
The target domain is `fries.htb`, and the attack chain begins with identifying open services through aggressive nmap scans, which reveal ports such as 22 (SSH), 53 (DNS), 80 (HTTP), 88 (Kerberos), 135 (RPC), 389 (LDAP), 443 (HTTPS), 445 (SMB), 5985 (WinRM), and others indicative of a Windows Active Directory environment.

## DNS
Initial reconnaissance involves adding domain hostnames to the /etc/hosts file to resolve subdomains like `code.fries.htb, db-mgmt05.fries.htb, and pwm.fries.htb`, enabling access to hosted web applications.

## Gitea
A Gitea repository at `code.fries.htb` is discovered, which contains application code and reveals sensitive credentials, including a database URL with a password: `D4LE11maan!!`. This leads to the exploitation of a pgAdmin instance (db-mgmt05.fries.htb) vulnerable to CVE-2025-2945, allowing remote code execution and a shell as the pgadmin user inside a Docker container.

## Docker

From the compromised container, the attacker pivots further using Ligolo-ng to establish a tunnel to the internal network, gaining access to the Docker network (172.18.0.0/24) and the internal network (192.168.100.0/24). Linpeas reveals additional credentials, including a pgAdmin default password: Friesf00Ds2025!!. A writable /etc/passwd file and an NFS share mounted from 172.18.0.1 are discovered, with the share located at /srv/web.fries.htb and accessible to the public. By exploiting NFS misconfigurations and creating matching user and group IDs on the attacker machine, a SUID-enabled bash binary is placed on the shared NFS folder, allowing privilege escalation to the user barman on the web server.

SSH access is obtained as the svc user using the password Friesf00Ds2025!!, and further enumeration reveals the presence of Active Directory Certificate Services (AD CS) with a vulnerable template named "FryAuth". By leveraging the ESC1 (Certificate Authority trust) vulnerability, an attacker can request a certificate for a privileged user (e.g., Administrator@fries.htb) and use it to authenticate via PKINIT, resulting in the recovery of the NTLM hash of the Domain Administrator. This hash can then be used to authenticate and gain domain administrator access.

Further exploitation involves abusing Kerberos delegation through the msDS-AllowedToActOnBehalfOfOtherIdentity attribute, allowing the attacker to impersonate the Administrator on a target machine (FRIES-WEB) using S4U2Self and S4U2Proxy attacks, ultimately leading to full domain compromise. The attack chain demonstrates the use of multiple vulnerabilities across web applications, container escape, NFS misconfigurations, and Active Directory privilege escalation techniques.