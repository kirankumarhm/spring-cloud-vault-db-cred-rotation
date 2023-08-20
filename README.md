## Spring Boot Vault Demo

### Install Postgres
Follow instructions 
For Ubuntu - https://www.postgresql.org/download/linux/ubuntu/
For Windows - https://www.postgresqltutorial.com/install-postgresql/

- Find your pg_hba.conf, usually located under C:\Program Files\PostgreSQL\9.1\data\pg_hba.conf
- If necessary, set the permissions on it so that you can modify it. Your user account might not be able to do so until you use the security tab in the properties dialog to give yourself that right by using an admin override.
- Alternately, find notepad or notepad++ in your start menu, right click, choose "Run as administrator", then use File->Open to open pg_hba.conf that way.
- Edit it to set the "host" line for user "postgres" on host "127.0.0.1/32" to "trust". You can add the line if it isn't there; just insert host all postgres 127.0.0.1/32 trust before any other lines. (You can ignore comments, lines beginning with #).
- Restart the PostgreSQL service from the Services control panel (start->run->services.msc)

![image](https://github.com/kirankumarhm/spring-cloud-vault-db-cred-rotation/assets/6735087/48c7d468-d8ac-481e-a7f3-e203b374ff0b)


### Setup admin user in Postgres
```
postgres=# create user admin password 'admin123';
CREATE ROLE
postgres=# ALTER USER admin WITH SUPERUSER;
ALTER ROLE
postgres=# ALTER ROLE admin WITH LOGIN;
ALTER ROLE
postgres=# \q
```

### Install Hashicorp Vault
```
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install vault hashicorp/vault --set server.dev.enabled=true
```

#### Error on Windows
Get "https://127.0.0.1:8200/v1/sys/seal-status": http: server gave HTTP response to HTTPS client

#### Solution
Open Git Bash on Windows machine (if you have installed vault on windows), otherwise these commands will not work!!!

### Vault init and unseal
```
vault operator init
vault operator unseal <<keys 1>>
vault operator unseal <<keys 2>>
vault operator unseal <<keys 3>>
```

```
export VAULT_ADDR='http://localhost:8200'
export VAULT_TOKEN=hvs.OoYTWLHCJDAfEortz7ZroWS2
```


### Enable the database secrets engine
```
vault secrets enable database
```

### Configure PostgreSQL secrets engine
```
vault write database/config/postgresql \
     plugin_name=postgresql-database-plugin \
     connection_url="postgresql://{{username}}:{{password}}@host.docker.internal:5432/postgres?sslmode=disable" \
     allowed_roles="*" \
     username="admin" \
     password="admin123"
```

### Verify the configuration
```
vault write database/roles/myrole db_name=postgresql \
     creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT USAGE ON ALL SEQUENCES IN SCHEMA public TO \"{{name}}\"; GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
     default_ttl="30s" \
     max_ttl="1m"
```
