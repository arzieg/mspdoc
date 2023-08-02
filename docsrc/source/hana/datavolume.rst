.. _hana_datavolume:

##########################
HANA Datavolumemanagement
##########################

How to reclaim fragmented space at OS level from HANA Data volume
******************************************************************

`https://blogs.sap.com/2020/09/11/how-to-reclaim-fragmented-space-at-os-level-from-hana-data-volume/`

1. ```select * from  SYS.M_VOLUME_FILES WHERE FILE_TYPE = 'DATA'```
2a. ALTER SYSTEM RECLAIM DATAVOLUME 120 DEFRAGMENT; = >Our final data volume size will be payload + 20% fragmented space =>Can be used normally

If we want to run defragment only on specific node the we can use the below command:
2b. ALTER SYSTEM RECLAIM DATAVOLUME '<node server name>:3<nn>03' 120 DEFRAGMENT; =>Our final data volume size will be payload + 20% fragmented space =>Can be used normally

If the fragmentation percentage is really huge, it will take a lot of time and performance issue if we try to defragment our data volume to 120% . In this case , 
it is recommended to run the same command in pieces like below . 
    alter system reclaim datavolume 150 DEFRAGMENT
    alter system reclaim datavolume 140 DEFRAGMENT
    ...

Vorgehen bei Systemreplication, wie umgehen mit der zweiten Seite: `2348397 - Datenvolumen auf Sekundärseite mit SAP-HANA-Systemreplikation verkleinern`

How to Reclaim size of volume “/hana/log” when it is full.
************************************************************
`https://blogs.sap.com/2019/01/28/how-to-reclaim-size-of-volume-hanalog-when-it-is-full./`

Das ist für die SYSTEMDB als auch für den TENANT durchzuführen!:

    ```ALTER SYSTEM RECLAIM LOG;```

View: 

.. code-block:: sql

    select top 1000 * from "SYS"."M_LOG_SEGMENTS"

    select b.host, b.service_name, a.state, count(*) 
    from "PUBLIC"."M_LOG_SEGMENTS" a 
    join "PUBLIC"."M_SERVICES" b on (a.host = b.host AND a.port = b.port) 
    group by b.host, b.service_name, a.state;


Status:
    Writing         Currently writing to the current log segment
    Closed          The segment is closed by the writer
    Truncated       but not yet backed up. Backup will remove it.
    BackedUp        Segment is already backed up, but a safepoint has not yet been written. Therefor it needs to be kept for instance recovery
    Free            Segment is free and can be reused (nur dann schafft reclaim log Freiplatz)

https://launchpad.support.sap.com/#/notes/1679938  Disk full Event on Log volume
https://launchpad.support.sap.com/#/notes/0002409471 Log Volume full on secondary site with system replication

When log volume is 100% full, it is a hung situation. DB will not allow any connections. So alter system reclaim log will not work.

1. stop the database
2. move few of the log segments to new location and create the soft link
3. start the database
4. check the status of the log segments.
5. select status from m_log_segments.
6. if the status of the log segments is 'free'  then execute 'alter system reclaim log'  to clear the space. i.e alter system reclaim log works only if the status of the log segment is ' free'
7. if the status is not free, then check the log backups if they are working fine.

