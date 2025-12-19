## Enumerate Public Repositories (FIRST)

#Attakiing_machine 
Open in browser:

```
http://code.fries.htb/explore/repos
```

![[repositories.png]]

You have **Unauthenticated access:** Enabled (`Explore` tab is visible).

## Enumerate users (this always works)

Open:
```
http://code.fries.htb/explore/users`
```

![[users.png]]

Great — those two users are **very important**.
### What you have

- `dale` (Dale Cooper) → likely a **real domain user**
    
- `administrator` → **huge red flag** (often reused, misconfigured, or protected but still useful)
    

This confirms:

- Gitea has **at least one normal dev user**
- An **Administrator account exists in the app context**
- Usernames are now valid targets for **AD / Kerberos attacks**

### Enumerate users via API

#Attakiing_machine 

```
curl http://code.fries.htb/api/v1/users/search?q=a 
```

![[api_users.png]]

If it returns users → **user enumeration confirmed**.

#### 
What the API output tells us (important)

From:
```
"active": false,
"last_login": "0001-01-01T00:00:00Z", 
"is_admin": false`
```

For **both** users (`administrator`, `dale`):

### ✅ Key observations

1. **Users exist** ✔️
2. **They have NEVER logged in**
3. **Accounts are inactive**
4. **Not local Gitea admins**
5. Emails are `@noreply.localhost`

👉 This strongly indicates:

## 🔥 **Gitea is using EXTERNAL AUTH (LDAP / AD)**

Users are **pre-created** but authenticate against `Active Directory`.

This matches perfectly with:
- LDAP SRV records
- Kerberos SRV records
- AD DC at `<MACHINE IP>`

**Next step:**  [[REAL attack paths]]
