#Attakiing_machine 
1. Check auth sources:

```
curl http://code.fries.htb/api/v1/admin/auths
```

If forbidden → expected.

2. Now check config leak:

```
curl http://code.fries.htb/api/v1/version
```

![[version.png]]

3. And:

```
curl http://code.fries.htb/api/swagger
```

![[swagger-1.png]]

Good, this is exactly what I expected 👍  
Let’s interpret what `/api/swagger` gives you and what it _doesn’t_, and then move forward cleanly.

#### What `/api/swagger` actually tells us

The HTML you pasted is just the **Swagger UI loader**:

`<div id="swagger-ui" data-source="/swagger.v1.json"></div>`

This means:

- ✅ The API **is fully documented**
- ❌ No auth config is leaked here
- ❌ No LDAP bind DN / password exposed
- ❌ No misconfig in the UI itself

So this endpoint is **informational**, not exploitable by itself.

The real value is the **API behavior you already exploited**:

- `/api/v1/users/search`
- `/api/v1/users`
- User enumeration without auth ✔️

That confirms:

> [!Note]
> **Gitea is intentionally exposed as a user directory, backed by AD**

## Confirmed Architecture (important)

At this point, we are 100% sure the setup is:

![[arquitecture.png]]

You already proved:

- LDAP requires bind ❌
- Kerberos exists ✔️
- Gitea users are AD-backed ✔️
- Users never logged in ✔️

So the **ONLY missing piece is credentials**.

---

### 🎯 What is NOT worth doing anymore

Don’t waste time on:

- Swagger UI fuzzing ❌
- CSRF token reuse ❌ (token is session-bound)
- API POSTs ❌ (auth required)
- Repo enumeration ❌ (none exist)

HTB wants you **out of the web layer** and into **AD auth attacks**.


