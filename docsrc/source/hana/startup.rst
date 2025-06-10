.. _hana_startup:

##########################
HANA Startup
##########################


HANA Startup
=============

https://community.sap.com/t5/technology-blogs-by-sap/sap-hana-startup/ba-p/13419359
https://launchpad.support.sap.com/#/notes/2222217 (How to troubleshoot SAP HANA Startup time)

Startup Process

1. Opening the Volumes
2. Loading and Initializing Persistence Structures
3. Loading or Reattaching the Row Store
4. Garbage-Collecting Versions
5. Replaying the Logs
6. Transaction Management
7. Savepoint
8. Checking the Row Store Consistency
9. Open SQL Port

ad 1) Opening the Volumes
    Nameserver mountet data und log Volumes
    Suche in nameserver.trc nach "DataVolumePartitionImpl.cpp"

ad 2) Loading and Initializing Persistence Structures
    
   * The anchor and the restart page are loaded. To HANA, these are something like the master boot record is to an operating system.

   * Converters and container directories are loaded.

   * Undo files are searched for information on transactions that have not committed before the system shut down. Don't be fooled by the fact that the traces refer to sessions rather than transactions – it’s uncommitted transactions.

   * HANA persistence statistics are loaded and initialized.
   
   * Package LOBs

   Suche nach: "AnchorPage", "PersistenceManag", "PersistenceSessionRegistry"

ad 3) Loading or Reattaching the Row Store (kann länger dauern)
    For as long as the system is up and running, the row store is kept in shared memory, owned by the corresponding HANA service operating system process. During shutdown, this 
    shared memory would normally be released back to the OS, while, during system startup, a significant part of the overall startup time would have to be spent on loading the 
    row store into memory again.

    Suche nach: "SmFastRestart", "CheckpointMgr", "SmFastRestart"

ad 4) Garbage-Collecting Versions

     HANA needs to do some cleanup for the column store as well. In this step, the garbage collector cleans up all version except for the most recent one for any column store table.
     With HANA 2, garbage-collecting obsolete versions is executed asynchronously. That means, it will allow the next step, replaying the logs, to continue in parallel.

     Suche nach: "PersistenceManagerImpl"

ad 5) Replaying the Logs

     HANA replays the logs to redo all changes that were performed after the last savepoint. After successfully finishing the replay, all uncommitted transactions are rolled back.

     Suche nach: "PersistenceManagerImpl", RecoveryHandlerImpl"

ad 6) Transaction Management

    Now, it's time that all services of a database synchronize with each other to ensure transactional consistency.

    Suche nach: "RowStoreTransactionCallback", "transmgmt"

ad 7) Savepoint

    All changes that have been performed in steps 3 - 5 are now persisted to the DATA volume by a savepoint.

    Suche nach: "PersistenceManagerImpl", "SavepointImpl"

ad 8) Checking the Row Store Consistency (max. 10 Min)

    If you haven't set parameter indexserver.ini -> [row_engine] -> consistency_check_at_startup to none, a row store consistency check is performed as last step during startup.

    Suche nach: "IntegrityChecker", "IntegrityCheckerTimer"

ad 9) Open SQL Port

    If you really see issues that cause opening the SQL port to be delayed or to fail, it's usually because another operating system process already occupies the port.

    Suche nach: "tcp_listener_callback", "TRexApiSystem"


Fehleranalyse: 
---------------

https://www.linkedin.com/pulse/hana-startup-issue-priyadarshi-amitav-jena

* The administrator can start looking at the trace files of different services, as per their starting sequence, to see if any error message can be found. i.e.,
  trace files for daemon, nameserver, indexserver, compile server, preprocessor etc. 

* During initial analysis, administrator can try restarting nameserver and indexserver manually by executing ./hdbnameserver and ./hdbindexserver from exe directory 
  in the sequence to see if these are starting successfully. In such case, issue could be with the daemon. Then admin can look further into the daemon trace file to find the 
  possible issue.

* A more generic startup issue could be an orphan process left out from the last shutdown, in such scenario, the admin must kill the process and remove the lock file 
  /usr/sap/<SID>/HDB<nn>/<hostname>/lock/hdb<servicename>@<port>.pid and restart. 



3203165 - Start scheitert bei Fehler nameserver: Coucould't find own nameserver in topology
---------------------------------------------------------------------------------------------

In diesem Fall: HANA SR wurde runtergefahren und ein `hdbnsutil -sr_cleanup -force` durchgeführt auf beiden Seiten. Auf der primären Seite startet die HANA, auf der
ehemaligen sekundären Seite startet HANA nicht. In diesem Fall der durchgeführt:

1. Stellen Sie sicher, dass das System vollständig heruntergefahren wird. Wenn ein Service noch ausgeführt wird, können Sie die Topologie beschädigen, und es gibt keine Möglichkeit, sie wiederherzustellen.
2. Führen Sie anschließend den folgenden Befehl als Benutzer <sid>adm im früheren Sekundärsystem aus:
hdbnsutil -exportTopology myExportedTopology.txt
3. Öffnen Sie die generierte Datei myExportedTopology.txt, und ersetzen Sie jeden Eintrag des Hostnamens des Primärstandorts durch den aktuellen Hostnamen des nicht registrierten Sekundärsystems.
4. Importieren Sie anschließend die korrigierte Topologiedatei mit dem folgenden Befehl im Zielsystem:
hdbnsutil -importTopology myExportedTopology.txt
5. Starten Sie nun die HANA-Instanz.
