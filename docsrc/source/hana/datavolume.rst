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

