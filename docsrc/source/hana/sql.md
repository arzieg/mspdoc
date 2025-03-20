# SQL

## hdbsql

```
hdbsql -n localhost -i 02 -u <user> -p <password> -d <dbsid>
hdbsql -i XX -u USERNAME -p PASSWORD -d systemDB

## Show Ports

```
SELECT * FROM SYS_DATABASES.M_SERVICES;

SELECT DISTINCT(sql_port) FROM SYS.M_SERVICES WHERE SQL_PORT > 0
```