## Enumerate Public Repositories (FIRST)

#Attakiing_machine 
Open in browser:

```
http://code.fries.htb
```

![[repositories.png]]

You have **Unauthenticated access:** Enabled (`Explore` tab is visible).

## Sign in

At the top-right you have a `Sign in` link.

![[signin.png]]

Introduce the credentials you have from the machine introduction:

Username: [d.cooper@fries.htb](mailto:d.cooper@fries.htb) 

Password: `D4LE11maan!!`

You'll see the following page, containing the repositories.

![[repos.png]]


## Gitea Instance (code.fries.htb)

Accessing `http://code.fries.htb` and reading the *README.md* file reveals a Gitea repository hosting application code.

**Key Intelligence from Repository:**

1. Flask application with PostgreSQL backend
2. Database management interface at `db-mgmt05.fries.htb`
3. Database name: `ps_db`
4. Key personnel: Dylan, Mike, Dale (infrastructure access)
5. Contact: `d.cooper@fries.htb`


### /etc/hosts

Add `db-mgmt05.fries.htb` to `/etc/hosts`.


### Git History Analysis

Searching through Git history reveals sensitive credentials of `ps_db` database:

```
git clone http://code.fries.htb/dale/fries.htb.git
```

Credentials:
Username: [d.cooper@fries.htb](mailto:d.cooper@fries.htb) 
Password: `D4LE11maan!!`

```
cd app
git log -p --all | grep -i "postgresql\|postgres\|database_url\|sqlalchemy" -B 5 -A 5
```

![[postgr_credentials.png]]

> [!ps_db Credentials]
> DATABASE_URL=postgresql://root:PsqLR00tpaSS11@172.18.0.3:5432/ps_db
SECRET_KEY=y0st528wn1idjk3b9a


**Next step:** [[Subdomain db-mgmt05.fries.htb]]
