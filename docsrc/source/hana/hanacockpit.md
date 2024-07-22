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


# Fehler
xs logs -recent cockpit-admin-web-app
7/19/24 3:38:48.602 PM [APP/6-1] SYS   #2.0#2024 07 19 15:38:48:602#+02:00#ERROR#/Handler#####c1305b82-64fe-4110-b633-df3e47c96c8e####TvjukMxcv6epsDPUcSp1vlqB8rKuJKLL######lysqxfux#PLAIN##GET request to /login/callback?code=hfHDap7qeZHeEiW8dFLxNiXN3xfuRRCL completed with status 500 Could not authenticate with xsuaa: Could not obtain access token: request to authentication service at https://laszis161.lunarlab.edeka.net:39632/uaa-security/oauth/token failed, error: unexpected response from authentication service at https://laszis161.lunarlab.edeka.net:39632/uaa-security/oauth/token: status code: 502, reason: no reason provided, #
Lösung: 
SAP Note 2243019, Wenn man mit selbst signierten Zertifikaten arbeitet, kann man auch XSA reset-certificate eingeben. Das Zertifikat ist nach einem Jahr ungültig.