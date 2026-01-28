
## External access paths

|                                                   |                                                                                                                                                                                                                                                                                                                                    |
| ------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![[external_access_paths_diagram-1.png\|306x459]] | - **Reverse Proxy** at the top, handling inbound traffic<br>    <br>- **Firewall** layer controlling access<br>    <br>- An **Internal Network** feeding into four containers: Gitea, PgAdmin, WebApp, PWM<br>    <br>- A shared **Docker Network** beneath the containers<br>    <br>- The underlying **Host System** at the base |


## Docker network layout

| ![[network_diagram.png\|348x524]] | <br>**Schematic of a Docker network layout**<br><br>- A central *Docker Network block*<br>    <br>- Four containers: *Gitea, PgAdmin, WebApp, and PWM*<br>    <br>- An *Internal Network* layer connecting the containers<br>    <br>- A *Host System* at the base, representing the physical or virtual machine |
| --------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

## Container-to-container routing


| ![[Containerrs_diagram.png\|380x570]] | - Direct communication paths between containers like Gitea, PgAdmin, WebApp, and PWM<br>    <br>- A shared Internal Network layer enabling routing<br>    <br>- A labeled Container-to-Container Routing zone showing how containers exchange data<br>    <br>- The underlying Docker Network and Host System anchoring the setup |
| ------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|                                       |                                                                                                                                                                                                                                                                                                                                   |



## 🌐 **Docker Network**


```
Network: fries_default
Subnet: 172.18.0.0/16
Gateway: 172.18.0.1
```

Everything inside the application stack lives here.

# 📦 **Containers Overview**

| Container Name    | Image                | Internal IP | Exposed Ports | Purpose                                  |
| ----------------- | -------------------- | ----------- | ------------- | ---------------------------------------- |
| **ps_db**         | postgres:16          | 172.18.0.x  | 5432          | Main PostgreSQL database for the web app |
| **pgadmin**       | dpage/pgadmin4:9.1.0 | 172.18.0.3  | 5050          | PgAdmin interface (db-mgmt05.fries.htb)  |
| **gitea**         | gitea/gitea:1.22.6   | 172.18.0.x  | 3000          | Gitea instance (code.fries.htb)          |
| **pwm**           | pwm/pwm-webapp       | 172.18.0.x  | 8443          | PWM password manager (pwm.fries.htb)     |
| **fries-web**     | custom Python app    | 172.18.0.x  | 5000          | Main Fries web application               |
| **reverse-proxy** | nginx or traefik     | 172.18.0.x  | 80, 443       | Routes external domains to containers    |

_(IPs vary depending on container startup order, but all live in 172.18.0.0/16.)_

# 🔗 **Container Relationships**


| ![[Container_relations.png\|405x608]]  |                                                                                                                                                                                                                                                                                                                                           |
| -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![[Container_relations1.png\|408x612]] | - **Reverse proxy (80/443)** routing domains to:<br>    <br>    - **Fries-web** (5000)<br>        <br>    - **PWM Web App** (8443)<br>        <br>    - **Gitea** (3000)<br>        <br>- **LDAP queries** from PWM Web App to **Active Directory**<br>    <br>- **Fries-web** connecting to **PostgreSQL** (5432) and **PgAdmin** (5050) |





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