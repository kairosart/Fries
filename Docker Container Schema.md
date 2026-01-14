
## 🌐 **Docker Network**

Code

```
Network: fries_default
Subnet: 172.18.0.0/16
Gateway: 172.18.0.1
```

Everything inside the application stack lives here.

# 📦 **Containers Overview**

|Container Name|Image|Internal IP|Exposed Ports|Purpose|
|---|---|---|---|---|
|**ps_db**|postgres:16|172.18.0.x|5432|Main PostgreSQL database for the web app|
|**pgadmin**|dpage/pgadmin4:9.1.0|172.18.0.x|5050|PgAdmin interface (db-mgmt05.fries.htb)|
|**gitea**|gitea/gitea:1.22.6|172.18.0.x|3000|Gitea instance (code.fries.htb)|
|**pwm**|pwm/pwm-webapp|172.18.0.x|8443|PWM password manager (pwm.fries.htb)|
|**fries-web**|custom Python app|172.18.0.x|5000|Main Fries web application|
|**reverse-proxy**|nginx or traefik|172.18.0.x|80, 443|Routes external domains to containers|

_(IPs vary depending on container startup order, but all live in 172.18.0.0/16.)_

# 🔗 **Container Relationships**

Code

```
                 ┌──────────────────────────────┐
                 │ reverse-proxy (80/443)       │
                 │ routes domains → containers  │
                 └───────────────┬──────────────┘
                                 │
       ┌───────────────────────────┼──────────────────────────────┐
       │                           │                              │
       ▼                           ▼                              ▼
┌──────────────┐        ┌────────────────┐             ┌────────────────┐
│ fries-web    │        │ PWM Web App    │             │ Gitea          │
│ 5000         │        │ 8443           │             │ 3000           │
└──────┬───────┘        └──────┬─────────┘             └───────┬────────┘
       │                       │                               │
       │                       │ LDAP queries                  │
       │                       ▼                               │
       │                Active Directory                       │
       │                                                       │
       ▼                                                       ▼
┌──────────────┐                                        ┌────────────────┐
│ PostgreSQL   │◄────────────── fries-web ──────────────│ PgAdmin        │
│ 5432         │                                        │ 5050           │
└──────────────┘                                        └────────────────┘
```

# 🧩 **Host ↔ Container Interaction**

The host (192.168.100.2):

- Runs Docker Engine
    
- Mounts volumes into containers
    
- Exposes NFS export `/srv/web.fries.htb`
    
- Provides internal-only services to containers
    

Code

```
Host (192.168.100.2)
│
├── Docker Engine
│     ├── fries-web
│     ├── pwm
│     ├── gitea
│     ├── pgadmin
│     └── ps_db
│
└── NFS export → used for privilege escalation
```

# 🧠 **Key Observations for Attackers**

- **PWM** is not “just another container” — it’s the _LDAP injection entry point_.
    
- **ps_db** is the first internal pivot after credential extraction.
    
- **fries-web** and **gitea** leak additional secrets/configs.
    
- **pgadmin** exposes DB management but is only reachable internally.
    
- **reverse-proxy** is the only container exposed externally.
    
- The **host** is the real target because it exposes NFS with weak permissions.