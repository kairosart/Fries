# Fries
The HackTheBox machine "Fries" is a hard difficulty challenge that involves a multi-stage attack on a complex Active Directory infrastructure with interconnected systems, including a Windows Server 2022 Domain Controller and an Ubuntu web server. 

## Nmap
The target domain is `fries.htb`, and the attack chain begins with identifying open services through aggressive nmap scans, which reveal ports such as 22 (SSH), 53 (DNS), 80 (HTTP), 88 (Kerberos), 135 (RPC), 389 (LDAP), 443 (HTTPS), 445 (SMB), 5985 (WinRM), and others indicative of a Windows Active Directory environment.

## DNS
Initial reconnaissance involves adding domain hostnames to the /etc/hosts file to resolve subdomains like `code.fries.htb, db-mgmt05.fries.htb, and pwm.fries.htb`, enabling access to hosted web applications.

## Gitea
A Gitea repository at `code.fries.htb` is discovered, which contains application code and reveals sensitive credentials, including a database URL with a password: `D4LE11maan!!`. This leads to the exploitation of a pgAdmin instance (db-mgmt05.fries.htb) vulnerable to CVE-2025-2945, allowing remote code execution and a shell as the pgadmin user inside a Docker container.

## Docker

From the compromised container, the attacker pivots further using *Ligolo-ng* to establish a tunnel to the internal network, gaining access to the Docker network (172.18.0.0/24) and the internal network (192.168.100.0/24).

*Linpeas* reveals additional credentials, including a pgAdmin default password: Friesf00Ds2025!!. A writable /etc/passwd file and an NFS share mounted from 172.18.0.1 are discovered, with the share located at /srv/web.fries.htb and accessible to the public. By exploiting NFS misconfigurations and creating matching user and group IDs on the attacker machine, a SUID-enabled bash binary is placed on the shared NFS folder, allowing privilege escalation to the user barman on the web server.

## SSH

SSH access is obtained as the svc user using the password Friesf00Ds2025!!, and further enumeration reveals the presence of Active Directory Certificate Services (AD CS) with a vulnerable template named "FryAuth". By leveraging the ESC1 (Certificate Authority trust) vulnerability, an attacker can request a certificate for a privileged user (e.g., Administrator@fries.htb) and use it to authenticate via PKINIT, resulting in the recovery of the NTLM hash of the Domain Administrator. This hash can then be used to authenticate and gain domain administrator access.

## Kerberos

Further exploitation involves abusing Kerberos delegation through the msDS-AllowedToActOnBehalfOfOtherIdentity attribute, allowing the attacker to impersonate the Administrator on a target machine (FRIES-WEB) using S4U2Self and S4U2Proxy attacks, ultimately leading to full domain compromise. The attack chain demonstrates the use of multiple vulnerabilities across web applications, container escape, NFS misconfigurations, and Active Directory privilege escalation techniques.


---
![[Diagram_general.svg]]


### 🔁 Step‑by‑step pivot checklist

**1. Enumerate and loot Gitea**

- **Goal:** Get internal creds and service URLs.
    
- **Do:**
    
    - Log into `code.fries.htb` (Gitea) with provided HTB creds.
        
    - Browse repos for:
        
        - **PostgreSQL connection string**
            
        - **PgAdmin URL** (`db-mgmt05.fries.htb`)
            
        - **Reused credentials** (DB/PgAdmin).
            

**2. Log into PgAdmin**

- **Goal:** Turn leaked creds into authenticated access.
    
- **Do:**
    
    - Browse to `db-mgmt05.fries.htb`.
        
    - Log in with credentials from Gitea.
        
    - Confirm you can open the **Query Tool**.
        

**3. Exploit PgAdmin RCE**

- **Goal:** Get a shell inside a container.
    
- **Do:**
    
    - Use the PgAdmin RCE vector (e.g., Metasploit module or manual payload) to run commands.
        
    - Confirm you have a shell as `pgadmin` user in a container.
        
    - Enumerate env vars and filesystem.
        

**4. Extract svc credentials from environment**

- **Goal:** Turn container access into host access.
    
- **Do:**
    
    - Dump environment variables (`env`, `printenv`, etc.).
        
    - Identify a password like `Friesf00Ds2025!!`.
        
    - Use it with `ssh` against the Linux host to find the valid user (e.g., `svc`).
        

**5. SSH into the Linux host as svc**

- **Goal:** Land on the underlying host OS.
    
- **Do:**
    
    - SSH to the box with `svc@<target>` and the leaked password.
        
    - Confirm you’re on the **host**, not in a container.
        
    - Enumerate mounts and exports.
        

**6. Abuse NFS export**

- **Goal:** Reach Docker TLS material and escape the export.
    
- **Do:**
    
    - Mount the NFS export (e.g., `/srv/web.fries.htb`) locally.
        
    - Use UID spoofing / FUSE tooling to:
        
        - Escape the export boundary.
            
        - Read sensitive files (especially Docker client certs/keys).
            

**7. Use Docker TLS certs to control Docker API**

- **Goal:** Gain root‑equivalent control over containers.
    
- **Do:**
    
    - Configure Docker client with the stolen certs.
        
    - Talk to Docker API on port `2376`.
        
    - Verify you can `docker ps`, `docker run`, etc.
        

**8. Spawn your own container on app_net**

- **Goal:** Join the network where 192.168.100.2 lives.
    
- **Do:**
    
    - Identify the app network (e.g., `app_net`).
        
    - Run:
        
        - `docker run -it --network app_net alpine sh`
            
    - Inside the container:
        
        - `ip a` → confirm `192.168.100.x` address.
            

**9. Reach 192.168.100.2 (PWM)**

- **Goal:** Talk to PWM directly.
    
- **Do (inside app_net container):**
    
    - `ping 192.168.100.2` (optional).
        
    - `curl http://192.168.100.2` → confirm PWM web app.
        
    - Locate PWM config files (e.g., `PwmConfiguration.xml`).
        

**10. Modify PWM to redirect LDAP**

- **Goal:** Capture domain credentials.
    
- **Do:**
    
    - Edit PWM config to point LDAP to your controlled IP.
        
    - Restart/reload PWM if needed.
        
    - Run a listener (e.g., Responder‑style) to capture LDAP binds.
        

**11. Use captured domain creds for AD access**

- **Goal:** Turn captured creds into domain‑level visibility.
    
- **Do:**
    
    - Validate creds against LDAP/Kerberos.
        
    - Use them with BloodHound ingestors to map AD.
        
    - Proceed with whatever AD path the graph reveals.