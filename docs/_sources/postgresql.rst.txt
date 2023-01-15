.. _postgresql:

Konfiguration
###############

Remote Access ermöglichen
==========================
1. /etc/postgresql/14/main/postgresql.conf
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
    

2. /etc/postgresql/14/main/pg_hba.conf
   
    # Netzwerkbereich zulassen, der zugreifen darf
      host    all             all             <IP Netz>/24        md5

3. systemctl restart postgresql
4. lsof -i:5432
5. less /var/log/postgresql/postgresql-14-main.log
6. psql -h <IP> -p 5432 -d <DB> -U <user> -W



Zurück zu :ref:`postgresql`