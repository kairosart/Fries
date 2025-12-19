
#Attakiing_machine 
Look for subdomains.

### DNS ANY Query Summary – *fries.htb*

```
dig @<MACHINE IP> fries.htb ANY
```

![[dig1.png]]


#### Key Findings

- **Authoritative DNS server:** `dc01.fries.htb`
    
- **Domain Controller:** `dc01.fries.htb` = **10.10.11.96** (also **192.168.100.1** internally)
    
- **Domain resolves to multiple IPs:**
    
    - IPv4: `10.10.11.96`, `192.168.100.1`
        
    - IPv6: `dead:beef::d950:6007:5d0:916a` (likely lab / non-production)
        

#### Records Returned

- **A records:** External HTB IP + internal LAN IP
    
- **NS record:** `dc01.fries.htb`
    
- **SOA record:** Confirms AD-integrated DNS
    
- **AAAA record:** Present (IPv6 enabled)
    

#### Security / Attack Surface Implications

- **Single DNS + DC:** DNS and AD hosted on the same machine (`DC01`)
    
- **Internal IP leak:** `192.168.100.1` suggests **dual-homed DC** (internal network exposure)
    
- **IPv6 enabled:** Possible attack vector (many AD environments overlook IPv6 security)
    
- **Authoritative responses:** Confirms DNS is AD-integrated and reachable


### LDAP SRV Query Summary – _ldap._tcp.fries.htb`

```
dig @10.10.11.96 _ldap._tcp.fries.htb SRV
```

![[ldap.png]]


What this Confirms

- **LDAP service location:**
    
    - Host: `dc01.fries.htb`
        
    - Port: **389**
        
    - Priority: **0** (highest)
        
    - Weight: **100**
        
- **Single Domain Controller:**  
    All LDAP traffic for `fries.htb` points to **DC01**.
    

### Kerberos SRV Query Summary – _kerberos._tcp.fries.htb

```
 dig @<MACHINE IP> _kerberos._tcp.fries.htb SRV
```

![[kerveros.png]]

#### What This Confirms

- **Kerberos KDC:** `dc01.fries.htb`
    
- **Port:** `88/tcp`
    
- **Priority / Weight:** `0 / 100`
    
- **Single Domain Controller:** All Kerberos auth is handled by **DC01**.
    

##### Infrastructure Picture (Now Complete)

You now have confirmation of:

- **DNS:** dc01.fries.htb
    
- **LDAP:** dc01.fries.htb :389
    
- **Kerberos:** dc01.fries.htb :88
    
- **SMB / RPC / WinRM:** same host
    

➡️ This is a **clean, single-DC Active Directory lab**, exactly what HTB likes.
##### Infrastructure Details

- **DC01 IPs:**
    
    - External: `10.10.11.96`
        
    - Internal: `192.168.100.1`
        
    - IPv6: `dead:beef::d950:6007:5d0:916a`
        
- **Authoritative response:** AD-integrated DNS is functioning normally.
    

##### Security / Enumeration Implications

- **No redundancy:** One DC = single point of failure and single attack focus.
    
- **LDAP exposed (389):**
    
    - Test **anonymous / unauthenticated LDAP binds**
        
    - Enumerate domain info, users, groups if misconfigured
        
- **Clear AD layout:** Simplifies Kerberos, LDAP, and SMB enumeration.





---
## Immediate Web-Focused Action Items

Visit & fingerprint.

```
whatweb http://fries.htb
```

![[whatweb.png]]


### WhatWeb Result Summary – `http://fries.htb`

**Status:** `200 OK`  
**Web Server:** `nginx/1.18.0 (Ubuntu)`  
**Tech Stack:**

- **HTML5**
    
- **Bootstrap**
    
- **Client-side scripting (JS)**
    
- **No CMS detected** (important)
    

**Metadata Discovered:**

- **Title:** _“Welcome to Fries – Fries Restaurant”_
    
- **Contact email:** `info@fries.htb`
    
- **Country:** `ZZ` (placeholder / non-configured)
    

### What This Tells Us (Security-Wise)

✅ Likely a Custom Web App
No WordPress, Joomla, Drupal, etc. detected →  

➡️ This is almost certainly a **custom-built site**, which is **good news for attackers**.

✅ Valid Email Pattern

- `info@fries.htb` confirms:
    
    - Domain: `fries.htb`
        
    - Email format: `*@fries.htb`
        

This helps with:

- **Username guessing**
    
- **Password reset abuse**
    
- **Later Kerberos user enumeration**
    

#### ⚠️ Static Front Page ≠ Safe Backend

Restaurant-style homepage is often:

- Just a landing page
    
- Backend functionality hidden elsewhere:
    
    - `/admin`
        
    - `/login`
        
    - `/staff`
        
    - `/portal`
        
    - API routes

❌ There aren't any of those backends.


---

## Directory brute-force `fries.htb`

#Attakiing_machine 
Do a VHost brute-force.

```
ffuf -u http://<MACHINE IP>/ -H "Host: FUZZ.fries.htb" \
     -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt \
     -fs 154
```

![[ffuf.png]]

- You’ve discovered a **new virtual host**: `code.fries.htb`.
    
- Very likely a **dev / source / internal app**.
    
- HTB almost always hides **credentials or logic flaws** here.


## Do This Immediately (In Order)

 1. Open `/etc/hosts`.

```
sudo nano /etc/hosts
```

2. Add it to `/etc/hosts` .
```
<MACHINE IP> code.fries.htb
```

3. Pull the Page Source (Don’t Trust the Browser Alone)

```
curl -s http://code.fries.htb | tee code.html
```

This will give you a file called `code.html` with the whole code.

Perfect — this confirms  `code.fries.htb`  is a public Gitea instance (v1.22.6)**. This is a high-value foothold. Here’s exactly how to proceed, step by step, based on what you’ve shown.

![[code.fries.htb.png]]


## What we know (important)

- **Service:** Gitea **1.22.6**
    
- **Unauthenticated access:** Enabled (Explore is visible)
    
- **API available:** `/api/swagger`
    
- **Likely dev repos:** This is where creds usually leak
    
- **HTB pattern:** Gitea → repo → config → AD creds → WinRM/LDAP

**Next step:** [[Gitea Public repositories]]
