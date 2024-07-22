.. _hana_cockpit:

# HANA Cockpit

## xs

xs login   -> einloggen auf OS Ebene

https://help.sap.com/docs/SAP_HANA_PLATFORM/6b94445c94ae495c83a19646e7c3fd56/6696e4fb6fcd4166b78a5a8432107a25.html
xs org     -> welche Org Einheiten gibt es 
xs spaces -> welche Schema/Spaces sind definiert
xs target -o HANACockpit -s SAP   -> Scope setzen
xs apps   -> zeige die Apps an in diesem Scope (um die URL z.B. zu erfahren)
xs logs --recent|--all <app>   -> zeige logs


### Self Signed Ceritificate neu erstellen
https://me.sap.com/notes/0002986229

Als sidadm 

```
cd $SECUDIR

Führen Sie den folgenden Befehl aus, um ein selbstsigniertes Zertifikat zu generieren (ab jetzt ca. 17 Jahre verfügbar).
sapgenpse gen_pse -p <pse_name> -noreq <Unterscheidungsname>

Zum Beispiel: 
sapgenpse gen_pse -p test.pse -noreq "CN=lssjc0007.sjc.sap.corp,OU=test_organization_unit,O=test_organization,C=DE"

Hinweis:
* Führen Sie "sapgenpse gen_pse -h" für weitere Details aus.
* CN ist für den Unterscheidungsnamen erforderlich. Legen Sie den Cockpit-FQDN-Namen als Wert fest.
* Die Werte OU (Organisationseinheit) und O (Organisation) können sich auf den entsprechenden Wert des selbstsignierten Standardzertifikats beziehen.
* Wenn Sie zur Eingabe von "PSE PIN/Passphrase" aufgefordert werden, drücken Sie die Eingabetaste, um die leere PIN festzulegen.
* Sie können den folgenden Befehl ausführen, um die Basisinformationen der neu generierten PSE-Datei zu prüfen:
   sapgenpse get_my_name -p test.pse -v

Führen Sie den folgenden Befehl aus, um den Domänennamen abzurufen:
        xs domains

Führen Sie den folgenden Befehl aus, um die neue PSE-Datei in die XSA-Domäne zu setzen:

xs set-certificate <Domänenname> --pse test.pse --pse-pin ""

oder
        xs set-certificate <Domänenname> --pse test.pse

Hinweis:
* Wenn Sie nach einer PIN fragen, drücken Sie direkt die Eingabetaste.
* Nachdem Sie das neue Zertifikat eingerichtet haben, können Sie den folgenden Befehl ausführen, um zu prüfen, ob das Domänenzertifikat aktualisiert wurde:
   xs-Domänenzertifikate

Führen Sie den folgenden Befehl aus, um XSA neu zu starten:
        xs restart -s SAP


Hinweis:
* Warten Sie einige Minuten nach dem Neustart, um sicherzustellen, dass die XS-Anwendungen neu gestartet wurden.
* Sie können den Befehl "xs a" ausführen und dann in der Spalte für die Instanzen der Anwendungen prüfen, ob die Anwendungen gestartet wurden. "1/1" bedeutet, dass die Anwendung gestartet wurde, "0/1" bedeutet, dass die Anwendung noch nicht gestartet wurde.
```



## XSA

XSA diagnose - run self diagnose
XSA reset-certificat  - recreate certificate


# Fehler
xs logs -recent cockpit-admin-web-app
7/19/24 3:38:48.602 PM [APP/6-1] SYS   #2.0#2024 07 19 15:38:48:602#+02:00#ERROR#/Handler#####c1305b82-64fe-4110-b633-df3e47c96c8e####TvjukMxcv6epsDPUcSp1vlqB8rKuJKLL######lysqxfux#PLAIN##GET request to /login/callback?code=hfHDap7qeZHeEiW8dFLxNiXN3xfuRRCL completed with status 500 Could not authenticate with xsuaa: Could not obtain access token: request to authentication service at https://laszis161.lunarlab.edeka.net:39632/uaa-security/oauth/token failed, error: unexpected response from authentication service at https://laszis161.lunarlab.edeka.net:39632/uaa-security/oauth/token: status code: 502, reason: no reason provided, #
Lösung: 
SAP Note 2243019, Wenn man mit selbst signierten Zertifikaten arbeitet, kann man auch XSA reset-certificate eingeben. Das Zertifikat ist nach einem Jahr ungültig.