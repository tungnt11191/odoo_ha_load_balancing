
# PgBouncer Configuration for Amtech Odoo Database

## 1. Introduction
Configuring PgBouncer for optimal performance with an Odoo database depends on your specific use case, the expected load, and the available resources. Below is a general guide to the best practices for configuring PgBouncer to work efficiently with Odoo.

## 2. Installation PgBouncer

Install PgBouncer via docker compose:

### a. Directory Structure:
```bash
your_project_directory/
├── docker-compose.yml
└── pgbouncer/
    ├── pgbouncer.ini
    └── userlist.txt
```

### b. docker-compose.yml:
```bash
version: '3.8'

services:
  pgbouncer:
    image: edoburu/pgbouncer
    container_name: pgbouncer
    volumes:
      - ./pgbouncer/pgbouncer.ini:/etc/pgbouncer/pgbouncer.ini
      - ./pgbouncer/userlist.txt:/etc/pgbouncer/userlist.txt
    ports:
      - "6432:6432"
    networks:
      - pg_network

networks:
  pg_network:
    driver: bridge
```

### c. pgbouncer.ini
```bash
[databases]
postgres = host=database_host_ip port=5432 user=databse_user dbname=postgres
amtech = host=database_host_ip port=5432 user=databse_user dbname=amtech

[pgbouncer]
listen_addr = 0.0.0.0
listen_port = 6432
auth_type = scram-sha-256 
auth_file = /etc/pgbouncer/userlist.txt
logfile = /var/log/pgbouncer/pgbouncer.log
pidfile = /var/run/pgbouncer/pgbouncer.pid
admin_users = postgres, odoo
pool_mode = session
max_client_conn = 1000
default_pool_size = 20
min_pool_size = 10
server_idle_timeout = 600
server_lifetime = 3600
query_timeout = 300
log_connections = 1
log_disconnections = 1
log_pooler_errors = 1
```

- **admin_users**
The list of users who have access to the PgBouncer administrative console to execute management commands like SHOW POOLS, SHOW CLIENTS.
- **pool_mode = session**
The connection pooling mode that PgBouncer will use. session keeps the connection open for the duration of the user session. This mode is suitable for Odoo as it often maintains session state.
- **default_pool_size**
Set the pool size according to the number of concurrent connections your Odoo instance needs.
The pool size should be set based on the number of database connections your server can handle effectively, usually no more than 5 to 10 connections per CPU core.
- **max_client_conn**
This is the maximum number of client connections PgBouncer will accept. Set this higher than default_pool_size to allow for bursts of activity.
- **min_pool_size**
Ensures that a minimum number of connections remain open in the pool to handle sudden spikes in traffic.
- **server_idle_timeout**
The maximum time (in seconds) that a connection can remain idle before being closed.
- **server_lifetime**
The maximum lifetime (in seconds) of a server connection before it is closed and reopened. This setting helps recycle connections to avoid long-lived connections that could lead to resource exhaustion
- **query_timeout**
The maximum time (in seconds) that a query can run before it is terminated by PgBouncer..



### d. userlist.txt
```bash
"databse_user" "databse_user_password"
"postgres" "pgbouncerpostgrespassword"
```

### e. Start PgBouncer
```bash
docker-compose up -d
```

### g. Test the Connection

```bash
docker-compose up -d
```

**Connect to PgBouncer:**
```bash
psql -h localhost -p 6432 -U databse_user -d amtech
```

**Check PgBouncer status:**

```bash
SHOW clients;
SHOW pools;
```


## 3. Notes
- Make sure to configure PostgreSQL to accept connections from PgBouncer by updating the `pg_hba.conf` file.
    ```bash
    host    all             all             0.0.0.0/0               md5
    ```
- Monitor PgBouncer logs regularly to identify and troubleshoot any connection issues.
- Adjust the PgBouncer configuration according to your system's performance and usage patterns.
