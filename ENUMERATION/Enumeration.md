## Nmap
This a enumeration task. You can use *nmap* to figure out the answer.

#Attacking_machine

```
nmap -sC -sV <MACHINE ip>
```

![[nmap1.png|1242x730]]
![[NMAP2 1.png]]

## **Summary of Nmap Scan**

- **Host Role:** Appears to be a **Windows Active Directory Domain Controller** (`DC01.fries.htb`) with some **Linux-based web services**.

- **Domain:** `fries.htb` (AD domain), hostname `DC01`.


### Key Services

- **SSH (22/tcp):** OpenSSH 8.9p1 on Ubuntu.
    
- **DNS (53/tcp):** Simple DNS Plus.
    
- **Web (80, 443/tcp):**
    
    - Nginx 1.18.0 (Ubuntu).
        
    - HTTP redirects to `http://fries.htb/`.
        
    - HTTPS certificate for `pwm.fries.htb` (Fries Foods LTD).
        
- **Kerberos (88/tcp):** Windows Kerberos (confirms AD).
    
- **RPC / SMB (135, 139, 445/tcp):** Windows RPC & SMB; SMB signing required.
    
- **LDAP / AD (389, 636, 3268, 3269/tcp):**
    
    - Active Directory LDAP & Global Catalog (SSL and non-SSL).
        
    - Certificates reference `DC01.fries.htb`, `fries.htb`.
        
- **Kerberos Password (464/tcp):** kpasswd.
    
- **RPC over HTTP (593/tcp).**
    
- **WinRM (5985/tcp):** Microsoft HTTPAPI (likely Windows Remote Management).
    
- **Other:** 2179/tcp (vmrdp?).
    

### Notable Observations

- **Mixed OS footprint:** Linux (nginx, SSH) + Windows AD services.
    
- **Time skew:** ~**+7 hours** difference from scanner (important for Kerberos attacks).
    
- **SMB signing:** Enabled and required (limits some SMB relay attacks).
    
- **Certificates:** Long-lived AD certs; web cert valid 2025–2026.
    

### Overall Assessment

This host is a `Domain Controller for fries.htb` running standard AD services, with **external-facing web services on Ubuntu/nginx**. Likely attack surface includes **HTTP/HTTPS virtual hosts, AD/Kerberos, LDAP, and WinRM**, with Kerberos time skew being a potentially exploitable detail.


## Add domain to /etc/hosts

Before proceeding, we must ensure our attack machine can resolve the domain name. The LDAP service revealed the domain name: `fries.htb`.

#Attacking_machine 
Add the following line to the file `/etc/hosts`.

```
<MACHINE IP>    fries.htb  
```


**Next step:** [[Subdomains]]



