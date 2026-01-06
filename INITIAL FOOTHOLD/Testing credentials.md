With the architecture mapped,  move to exploitation. You have two entry points: 
- The Linux Web Tier (Port 80/22) 
- The Windows AD Tier (Port 445/88).

## Credential Verification

The engagement documentation provided specific credentials:

- **User:** `d.cooper@fries.htb`
- **Password:** `D4LE11maan!!`

## Testing SSH (The Linux Tier):

#Attacking_machine 
```
ssh d.cooper@<MACHINE IP>
```

Result: Permission denied (publickey).
The SSH port seems locked down to key-based authentication, or d.cooper is not a valid user on the Linux subsystem.


## Testing SMB (The Windows Tier)

Use *NetExec* (formerly CrackMapExec) to test the credentials against the Active Directory.

```
nxc smb <MACHINE IP> -u 'd.cooper' -p 'D4LE11maan!!'
```

![[nxc.png]]


> [!Success]
> The credentials are valid for the Active Directory environment.

### SMB Share Enumeration

#Attacking_machine 
With a valid AD session, you can enumerate the SMB shares. This is often the bridge between the two layers. *Developers frequently mount Windows shares into Linux web servers to access code or configuration files.*

