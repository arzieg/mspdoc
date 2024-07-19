# HANA Usermanagement

## hdbuserstore

hdbuserstore list           - definierte Keys im Userstore
hdbuserstore delete <Key>   - löschen eines Keys
hdbuserstore set            - Key definieren, Bsp: hdbuserstore set BACKUP lavdb10y03101:31013 BACKUPUSER <Passwort>


## Password reset


### Password neu setzen

Hier noch das Problem, dass SYSTEM sich nach dem Start nicht selber ändern darf.

```
/usr/sap/<SID>/HDB<instance>/exe/sapcontrol -nr <instance> -function StopSystem HDB
/usr/sap/<SID>/HDB<instance>/hdbenv.sh
/usr/sap/<SID>/HDB<instance>/exe/hdbnameserver -resetUserSystem
```

HANA starten und danach Passwort wieder neu setzen


### Sofern es einen User mit "USER ADMIN" Rechten gibt: 

```
# If you want to find all the users who have "USER ADMIN" privilege, please use below SQL:
SELECT GP.GRANTEE FROM GRANTED_PRIVILEGES GP WHERE GP.PRIVILEGE='USER ADMIN'
```

Dann mit dem User anmelden. 

```
ALTER USER SYSTEM DISABLE PASSWORD LIFETIME;
ALTER USER SYSTEM ACTIVATE USER NOW;
ALTER USER SYSTEM RESET CONNECT ATTEMPTS;
ALTER USER SYSTEM ENABLE CLIENT CONNECT;
--ALTER USER SYSTEM PASSWORD "password" NO FORCE_FIRST_PASSWORD_CHANGE;
CONNECT SYSTEM PASSWORD "password";
```



# HANA Ports


xs-admin-login

```
    Enter password for XSA admin user 'COCKPIT_ADMIN':

    API_URL: https://<hostname>:39630
    USERNAME: COCKPIT_ADMIN
    Authenticating...
    ORG: HANACockpit
    SPACE: SAP
    API endpoint:   https://<hostname>:39630 (API version: 1)
    User:           COCKPIT_ADMIN
    Org:            HANACockpit
    Space:          SAP
```

xs apps   - Anzeige aller Services und Ports


2986229 - Generieren eines selbstsignierten Zertifikats für die XSA-Domäne mit längeren Validierungstagen
    cd $SECUDIR
    sapgenpse gen_pse -p <PSE_Name> -noreq <Distinguish_Name>
    Bsp: sapgenpse gen_pse -p beispiel.pse -noreq "CN=<fqdn>,OU=test_organization_unit,O=test_organization,C=DE"
     sapgenpse get_my_name -p beispiel.pse -v

     xs domains           Anzeigen der Domänen 
     xs set-certificate <Domänenname> --pse beispiel.pse
        xs set-certificate <Domäne> --pse beispiel.pse

        Exporting PKCS8 from PSE...
        PIN>
        Setting SSL certificate for domain <domäne> as COCKPIT_ADMIN...
        OK
        TIP: Restart the SAP XS Controller to ensure your changes take effect for all applications.
        Alternatively use 'xs restage' and 'xs restart' for all applications.

    xs spaces
    xs restart -s <SPACE>


