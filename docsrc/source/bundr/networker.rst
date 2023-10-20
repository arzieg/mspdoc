.. _networker:

##########################
Networker Troubleshooting
##########################


NetWorker: Backup schlägt fehl mit „Authentifizierungsfehler; Grund = Server hat Zugangsdaten abgelehnt“
*********************************************************************************************************
`https://www.dell.com/support/kbdoc/de-de/000203002/backup-schlaegt-fehl-mit-authentifizierungsfehler-grund-server-hat-zugangsdaten-abgelehnt`

Ursache
-------
Peering-Zertifikate werden auf jedem NetWorker-System gespeichert. Probleme mit Peer-Informationen zwischen dem NetWorker-Client, dem Storage-Node und/oder dem NetWorker-Server 
führen zu diesem Fehler.

1. Auf dem Clientsystem

    ``` nsradmin -C -y -p nsrexecd "nsr peer information" ´´´

2. Peer-Informationen manuell löschen
   
.. code-block:: shell

    nsradmin -C

    nsradmin -p nsrexec
    nsradmin> p type: nsr peer information; name: <client_name>
    nsradmin> delete

    systemctl stop networker
    systemctl start networker
   
3. Auch auf dem NW-Server löschen: 

.. code-block:: shell

    nsradmin -s <server_name> -p nsrexec
    nsradmin> p type: nsr peer information; name: <client_name>
    nsradmin> delete

4. LOG File betrachten

    ```tail -f /nsr/applogs/hdbbackintY03.log.raw```

    