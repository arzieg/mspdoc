# RAC

https://docs.oracle.com/en/database/oracle/oracle-database/21/cwadd/oracle-clusterware-control-crsctl-utility-reference.html#GUID-5F74BE9A-1F85-4E60-B321-693FC70500A3

## Service

```
srvctl status service -d S77     # status service 
srvctl relocate service -d S77 -s S77HA -i S772 -t S771      # service umschalten von ... nach ...
srvctl relocate service -d S77 -s S77HA -i S772 -t S771 -f   # force 

srvctl start instance -d S77 -i S771   // Instanz nur auf Knoten 1 starten
srvctl stop instance -d s77 -i S772  [-stopoption TRANSACTIONAL LOCAL]  // Instanz auf Knoten 2 stoppen
srvctl start service -s S77HA -i S771 -db S77   // Start HA Server auf 1 Knoten

crsctl stat res -t -init         // Check Status Cluster lokal
crsctl stat res -t               // Check Status Cluster global
crsctl stat res -p               // Check Parameter Cluster

srvctl stop database -d S77      // Stop Datenbank als oracle user mit S77-environment
srvctl stop database -d S77  -stopoption TRANSACTIONAL
								// Using the SHUTDOWN TRANSACTIONAL command with the LOCAL option is useful to shut down a particular Oracle RAC database instance. 
									Transactions on other instances do not block this operation. If you omit the LOCAL option, then this operation waits until transactions on 
									all other instances that started before you ran the SHUTDOWN command either commit or rollback, which is a valid approach, if you intend to 
									shut down all instances of an Oracle RAC database
srvctl start database -d S77     // Stop Datenbank als oracle user mit S77-environment
srvctl status database -db S77
srvctl enable database -d S77	// aktiviere DB in Clusterconfig
```

### Verify that instance is running:

```
srvctl status service -db db_unique_name -service service_name

col INST_NUMBER format 99999
COL INST_NAME for a20
SELECT * FROM V$ACTIVE_INSTANCES;

GV$SESSION   = View mit globalen Sessions
SELECT SID, SERIAL#, INST_ID FROM GV$SESSION WHERE USERNAME='SAPR3';
```

## Cluster

```
crsctl stop cluster             // Stop des Clusters auf dem lokalen Knoten
crsctl stop crs                 // Stop crs auf lokalen Knoten
crsctl stop cluster -all		// Stop des Clusters auf allen Knoten inkl. der DB
crsctl start cluster -all		// Start der CRS auf allen Knoten
```

```
The CRS-4535 Cannot communicate with Cluster
	prüfen: ps -ef |grep crsd.bin
			crsctl stat res -t -init
			crsctl start res ora.crsd -init

	myhost:~ # crsctl stop crs
	CRS-4639: Could not contact Oracle High Availability Services
	CRS-4000: Command Stop failed, or completed with errors.
	myhost:~ # ps -ef|grep ohasd
	root      4524     1  0 Nov04 ?        00:00:00 /bin/sh /etc/init.d/init.ohasd run >/dev/null 2>&1 </dev/null
	myhost:~ # crsctl stop crs -f
	CRS-4639: Could not contact Oracle High Availability Services
	CRS-4000: Command Stop failed, or completed with errors.
	myhost:~ # crsctl start crs
```


### Resourcen anlegen

https://dbacentrals.blogspot.com/2017/11/oracle-rac-error-crs-2805-unable-to-part1.html

```
srvctl config network           // welche Netzwerke sind im Cluster konfiguriert
srvctl config nodeapps -a       // welche VIPs sind definiert
/oracle/grid/191000/bin/srvctl add vip -n laszdb11 -A laszdb11-vip.<fqdn>/255.255.255.0 -k 1   (die -k 1 ist die Netzwerknummer) // VIP Resource anlegen
srvctl remove vip -vip laszdb11-vip // wieder removen
srvctl start vip -vip laszdb11-vip

Weitere Ressourcen anlegen
appvipcfg create -network=1 -ip 10.10.191.227 -vipname=laszdb11-vip -user=root -failback=0    // neuen vip anlegen
crsctl start res laszdb11-vip   // start vip
crsctl delete res laszdb11-vip  // delete resource
```

## CDB/PDB:

ALTER PLUGGABLE DATABASE PDB_NAME CLOSE IMMEDIATE  // Stoppen einer PDB, check über srvctl status service


## Cluster Verify

Download: https://support.oracle.com/epmos/faces/PatchSearchResults?_afrLoop=113536042256749&searchdata=%3Ccontext+type%3D%22BASIC%22+search%3D%22%26lt%3BSearch%26gt%3B%26lt%3BFilter+name%3D%26quot%3Bpatch_number%26quot%3B+op%3D%26quot%3Bis%26quot%3B+value%3D%26quot%3B30839369%26quot%3B%2F%26gt%3B%26lt%3BFilter+name%3D%26quot%3Bexclude_superseded%26quot%3B+op%3D%26quot%3Bis%26quot%3B+value%3D%26quot%3Bfalse%26quot%3B%2F%26gt%3B%26lt%3B%2FSearch%26gt%3B%22%2F%3E&_afrWindowMode=0&_adf.ctrl-state=6rynnulxx_4
./cluvfy stage -post crsinst -allnodes


## ASM

```
set linesize 200;
select name, state from V$ASM_DISKGROUP;    -> Anzeige Diskgruppen
ALTER DISKGROUP data1 MOUNT;                -> Mounte diskgruppe

col name for a10
col percentage for 999.99
SELECT name, free_mb, total_mb, free_mb/total_mb*100 as percentage FROM v$asm_diskgroup;
```

## SPFILE:

liegt im ASM, wenn man mal was sucht oder ändern muss

```
sqlplus / as sysdba
create pfile='/tmp/init.ora' from spfile;
```


## LISTENER:

Im Cluster unter dem Environment ASM findet man die Cluster Listener
Jede Datenbank kann über init.ora Parameter einen Listenerwert zugeordnet werden. 
	Achtung, SAP-Hinweis 1915325 - Konfiguration des Oracle-Datenbankparameters 'LOCAL_LISTENER' nicht für RAC?
	Hinweis: https://launchpad.support.sap.com/#/notes/2470718

```
alter system set local_listener='myhost:1521' scope=spfile sid='S661';
alter system set local_listener='laszdb13:1521' scope=spfile sid='S662';
alter system set remote_listener='l6rac:1521' scope=spfile sid='*';

[oracle@myhost]grid-+ASM1$ lsnrctl status LISTENER
...

Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=LISTENER)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=1.1.1.2)(PORT=1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=1.1.1.3)(PORT=1521)))
Services Summary...
Service "+ASM" has 1 instance(s).
  Instance "+ASM1", status READY, has 1 handler(s) for this service...
Service "S77.world" has 1 instance(s).
  Instance "S771", status READY, has 1 handler(s) for this service...
Service "S77XDB.world" has 1 instance(s).
  Instance "S771", status READY, has 1 handler(s) for this service...
Service "S66" has 1 instance(s).
  Instance "S661", status READY, has 1 handler(s) for this service...
Service "S661" has 1 instance(s).                                              <--- hier dann z.B. Ausgabe auf dem ersten Knoten
  Instance "S661", status READY, has 1 handler(s) for this service...
The command completed successfully



srvctl status listener   			# status local listener
srvctl status scan_listener  		# status scan listener
srvctl config scan_listener        	# config vom scan listener
srvctl modify scan_listener -p TCP:1560  # ändern des ports
srvctl config listener -l LISTENER	# config des listener
srvctl modify listener -l LISTENER -p TCP:1560  # ändern des Ports des lokalen Listeners
srvctl stop scan_listener
srvctl stop listener
srvctl start listener
srvctl start scan_listener
```

Service registrieren: srvctl add service -db S77 -service S77HA -role PRIMARY -policy AUTOMATIC -dtp false -notification false -failovermethod BASIC -clbgoal LONG -rlbgoal NONE -tafpolicy BASIC -preferred S771 -available S772
Troubleshooting: https://blogs.oracle.com/coretec/post/was-geht-ab-sqlnet-tracing-erste-schritte-bei-der-fehlersuche



## Archiver Stuck:

	Neu setzen der Destination:
		ALTER SYSTEM ARCHIVE LOG STOP;
		ALTER SYSTEM SET LOG_ARCHIVE_DEST='woauchimmer';
		ALTER SYSTEM ARCHIVE LOG START;
	
	Labor / ASM
		su - oracle; asm - environment laden
		asmcmd
			lsdg
			find +S77_ARCH *
			cd <Verzeichnis>
			rm <datum>/* oder wenn man sich sicher ist rm */*
			lsdg
			
		su - oracle; S77 - environment
			rman
				connect target / 
				crosscheck archivelog all;
				list expired archivelog ALL;   -> zeigt noch einmal die fehlenden Archive an
				DELETE expired archivelog ALL;
				(!! nur in Laborumgebung: delete archivelog all;)
				
				
			sqlplus / as sysdba
				set linesize 200;
				SELECT name, free_mb, total_mb, free_mb/total_mb*100 "%" FROM v$asm_diskgroup;
				
				Größe der Recovery File Dest.
				--------------------------------
				set lines 100
				col name format a60
				select name
				, floor(space_limit / 1024 / 1024) "Size MB"
				, ceil(space_used / 1024 / 1024) "Used MB"
				from v$recovery_file_dest
				order by name
				/
				
				alter system set db_recovery_file_dest_size=600G;   -> ändern der Größe
				
			srvctl stop database -d S77
			srvctl status database -d S77
			sqlplus / as sysdba
			startup mount
			alter database archivelog
			shutdown 
			srvctl start database -d S77
				

Was auch ging: 
	Instanz unten
	sqlplus / as sysdba
	startup mount
	
	zweite Session: 
	im Environment der DB:  Archivsicherung anwerfen, je Instanz ein Script: rman cmdfile=/nsr/apps/scripts/arch_backup_qbd.rman
	
## Backup:

RMAN: (http://www.juliandyke.com/Research/RMAN/BackupCommand.php)

```
	BACKUP DATABASE;
	BACKUP ARCHIVELOG ALL;
	BACKUP DATABASE PLUS ARCHIVELOG;
	BACKUP CURRENT CONTROLFILE;
	BACKUP SPFILE;
```

Aktiven Status abfragen:
```
	set linesize 150
	column STATUS format a10
	col OUTPUT_BYTES_DISPLAY format a15
	select START_TIME,END_TIME,STATUS,INPUT_TYPE,ELAPSED_SECONDS,OUTPUT_BYTES_DISPLAY from v$rman_backup_job_details order by 1 asc;
```

Liste der Backups:
```
	connect target sys/xxxxxx@s01
	connect catalog rman/rman@rmanrep
	RMAN> spool log to /nsr/applogs/bu.txt
	RMAN> LIST BACKUP
	RMAN> spool log off;
	RMAN> spool log to /nsr/applogs/bu-summary.txt;
	RMAN> LIST BACKUP SUMMARY;
	RMAN> spool log off;
	RMAN> spool log to /nsr/applogs/bu-by-file.txt;
	RMAN> RMAN> LIST BACKUP BY FILE;
	RMAN> spool log off;
```


### Recoverpoint

https://mikedietrichde.com/2017/08/29/fallback-strategy-flashback-to-guaranteed-restore-points/

Setze der Flashback
```
		SQL> alter system set db_recovery_file_dest_size=30G;
		SQL> alter system set db_recovery_file_dest='+S77_ARCH';
```

altes Oracle Home laden

```
	SQL> select con_id, name, time, guarantee_flashback_database from v$restore_point order by 1,2;
		GRP_BEFORE_UPGRADE_19
	shutdown abort
	startup mount
	flashback database to restore point "GRP_BEFORE_UPGRADE_19";
	alter database open resetlogs;
	select comp_id, status from dba_registry;
```


## Tools: 

bbed - Oracle Support Block Editor, wird normalerweise nicht compiliert, unter Oracle 11 nicht mehr vorhanden aber nachinstallierbar (http://laurentleturgez.wordpress.com/2011/06/29/bbed-in-oracle-11g/)
  http://www.databasejournal.com/features/oracle/article.php/3835546/Installing-Oracle-Block-Browser-and-Editor-tool-bbed.htm
  

  
## Oracle Lizenzen

### Überprüfung, welche Lizenzen ab wann benutzt wurden

```
	SELECT
	  NAME,
	  DETECTED_USAGES,
	  CURRENTLY_USED,
	  FIRST_USAGE_DATE
	FROM
	  DBA_FEATURE_USAGE_STATISTICS
	WHERE
	  VERSION = (SELECT VERSION FROM V$INSTANCE) AND
	  (DETECTED_USAGES > 0 OR CURRENTLY_USED != 'FALSE');

```

  
	
	
	
	
## Default Passwörter
=====================
http://www.databasejournal.com/features/oracle/article.php/3708891/Eight-Ways-to-Hack-Oracle.htm

Username        Password
applsys         apps
ctxsys          change_on_install
dbsnmp          dbsnmp
outln           outln
owa             owa
perfstat        perfstat
scott           tiger
system          change_on_install
system          manager
sys             change_on_install
sys             manager


## Usermanagement: 
===============
https://www.ateam-oracle.com/avoiding-and-resetting-expired-passwords-in-oracle-databases

Unlock expired User ohne Passwort zu ändern!

Adjusting the password expiration policy
```
	select * from dba_profiles where resource_name = 'PASSWORD_LIFE_TIME';
	ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED;
```
		
Checking for problematic accounts

`select username,account_status from dba_users where account_status like '%EXPIRED%' or account_status like '%LOCKED%';`
		
Resetting of problematic accounts
    
Option 1: new Passwort

```
spool on;
set echo off;
set heading off;
set feedback off;
SET   SERVEROUTPUT  OFF;
spool unlock.sql;
select 'ALTER USER '|| USERNAME || ' account unlock;' from dba_users where ACCOUNT_STATUS like '%LOCKED%';
spool off;
@unlock.sql;
spool on;
set echo off;
set heading off;
set feedback off;
SET   SERVEROUTPUT  OFF;
spool pwchangen.sql;
select 'ALTER USER '|| USERNAME || ' identified by password1;' from dba_users 
where ACCOUNT_STATUS like '%EXPIRED%' or ACCOUNT_STATUS like '%LOCKED%';
spool off;
@pwchangen.sql;
```


Option 2: restore previous password
```
spool on;
set echo off;
set heading off;
set feedback off;
SET   SERVEROUTPUT  OFF;
spool unlock.sql;
select 'ALTER USER '|| USERNAME || ' account unlock;' from dba_users where ACCOUNT_STATUS like '%LOCKED%';
spool off;
@unlock.sql;
spool on;
set lines 300;
set echo off;
set heading off;
set feedback off;
SET   SERVEROUTPUT  OFF;
spool pwchangeo.sql;
select 'ALTER USER '|| USERNAME || ' identified by values ''' || spare4 || ''';' from dba_users,user$ 
where ACCOUNT_STATUS like '%EXPIRED%' and USERNAME=NAME;
spool off;
@pwchangeo.sql;
```

## Oracle Flashback: 

```
select flashback_on from v$tablespace      // Flashback aktiv? 
select flashback_archive_name,status from dba_flashback_archive;   // flashback status
alter database flashback off;              // Flashback off
```

## RMAN

set your ORACLE_SID to an ASM instance 
ASMCMD> cd +POC2FLASH/POC2DWH/ARCHIVELOG
using 'rm' command you can delete the directory. 

But don't forget to remove the archive log details from the control file or the RMAN catalog 
For this you need to connect to the target database using rman
rman target sys/<password> nocatalog 
rman>crosscheck archivelog all
rman>delete expired archivelog all

## Check_MK

Installierte Plugins
```
root@<host>[/usr/lib/check_mk_agent/plugins]# ./mk_oracle 
<<<oracle_dataguard_stats>>> 
<<<oracle_instance>>> 
<<<oracle_logswitches>>> 
<<<oracle_longactivesessions>>> 
<<<oracle_performance>>> 
<<<oracle_processes>>> 
<<<oracle_recovery_area>>> 
<<<oracle_recovery_status>>> 
<<<oracle_sessions>>> 
<<<oracle_undostat>>> 
<<<oracle_jobs>>> 
<<<oracle_locks>>> 
<<<oracle_resumable>>> 
 <<<oracle_rman>>> 
<<<oracle_tablespaces>>> 
<<<oracle_ts_quotas>>> 
<<<oracle_instance>>> 
<<<oracle_asm_diskgroup>>> 
ORA-99999 ORACLE_HOME for ORACLE_SID=ZAPPLPC2 not found or not existing!   
```

-> und von hier sucht er die Datenbanken. DBs die hier nicht enthalten sind werden in check_mk rot angezeigt.

```
root@<host>[/usr/lib/check_mk_agent/plugins]# 
cat /etc/oratab | grep ':N' 
+ASM2:/oracle/grid/19000:N 
DB1:/oracle/DB1/19000:N 
DB2:/oracle/DB2/19000:N 
DB3:/oracle/DB3/19000:N   
```

## Oracle ASM

löschen von ASM Disks: 		powermax oder vplex ausführen, dann oracleasm deletedisk $disk
wenn das löschen nicht geht, kann man probieren: 

```
ps aux (auf Spalte achten mit D, das sind die blockienrenden Prozesse)
losf | grep dm-XX (schreibt noch ein Prozess auf das Device?)
multipath -r  (reload des Mulitpath)
```
	
ansonsten manuell den Header löschen: 
	


## Oracle DISK Erweiterung

0. Im ASM Status

    ```
    set lin 1000
    col header_status form a12
    col mode_status form a12
    col path form a20
    col name form a15
    select header_status, mode_status,state,os_mb,total_mb,free_mb, name, path from v$asm_disk;
    ```

1. Prüfen mittels powermax

2. resize der Platten mit Multipath (Skript siehe unten) auf beiden RAC-Knoten   (als root):
              
              ## multipathd resize map < device-nr>

              ## die device-nr findet man mit vplex raus

              ## rescan-scsi-bus.sh -a -l -w (nur für neue Platten)

   Bsp. für PZ0 (als root) - Bitte die Diskgruppe anpassen (hier PZ0_DATA)

Resize Disks
```
	for map in $(vplex | grep -i PZ0_DATA | awk '{print $NF}'); do for disk in $(multipath -ll | grep -A8 $map | grep sd | awk '{print $3}'); do echo 1 > /sys/block/$disk/device/rescan; 
    multipathd resize map $map; done ; done 2>/dev/null
```
3. Die neue Größe prüfen  (als root): 

```
    powermax   oder
    lsblk | grep 36000097000029790044053303031464 <-> die ID der Platte
``` 

4. Die Platten auf beiden RAC-Knoten mit oracleasm scannen  (als root): oracleasm scandisks

4a. ASM Status

```
set lin 1000
col header_status form a12
col mode_status form a12
col path form a20
col name form a15
select header_status, mode_status,state,os_mb,total_mb,free_mb, name, path from v$asm_disk;
```


5. in der ASM - Umgebung (als oracle), auf einer Seite, den resize durchführen: (Achtung, da wo die MGMTDB läuft, muss die Disk vergrösstert sein)
    
    damit wird die  Größe genommen, die das Betriebssystem zurück meldet

    `SQL> alter diskgroup <DG_NAME> resize all;`

  oder alternativ, dann wird die  Größe genommen, die angegeben wird (mache ich nie!):

    `SQL> alter diskgroup <DG_NAME> resize all size 100G;`

5a. ASM Status

```
set lin 1000
col header_status form a12
col mode_status form a12
col path form a20
col name form a15
select header_status, mode_status,state,os_mb,total_mb,free_mb, name, path from v$asm_disk;
```


## Neuen Knoten wieder hinzufügen

Gedächtnisprorokoll, da kann noch einiges fehlen. Szenario: host kaputt, wurde neu deployed, keine RAC Software

Instanz aus der Konfiguration löschen
	oracle@host] S77-S771$ srvctl remove instance -db S77 -instance S772
	Remove instance from the database S77? (y/[n]) y

Reste ggfs löschen auf host
	host
	---------
	$ORACLE_HOME/deinstall/deinstall -local
	rm -r /oracle/base/oracle.ahf

	host:
	----------
	crsctl delete node -n <fqdn>
	host:/oracle/grid/19 # srvctl stop vip -vip host-vip.<fqdn>
	host:/oracle/grid/19 # srvctl remove vip -vip host-vip.<fqdn>
	Please confirm that you intend to remove the VIPs host-vip.<fqdn> (y/[n]) y

	Knoten wieder installieren, als Oracle/Grid
	---------------------------------------------
	/oracle/grid/19/addnode/addnode.sh -ignoreSysPrereqs -ignorePrereq -silent "CLUSTER_NEW_NODES={host.<fqdn>}" "CLUSTER_NEW_VIRTUAL_HOSTNAMES={host-vip.<fqdn>}"
	root.sh lief nicht erfolgreich durch, da keine Voting-Disks gefunden wurden (die aber da waren). Dann versucht nach https://www.dba-oracle.com/t_rac_add_node.htm:
		root $ cd $GRID_HOME/crs/install
		root $ ./rootcrs.sh -verbose -deconfig -force	
		root $ cd $GRID_HOME
		root $ ./root.sh
	Hat aber auch nicht so richtig geholfen. reboot des Systems und danach ein /oracle/grid/191700/root.sh - dann funktionierte es :-) 
	Dann anch Kapitel 6 die Arbeiten für die GRID Installation durchgeführt (bzw. welche notwendig waren)
	Dann noch Services registrieren
	srvctl add instance -db S77 -instance S772 -node host
	srvctl remove service -db S77 -service S77HA
	srvctl add service -db S77 -service S77HA -role PRIMARY -policy AUTOMATIC -dtp false -notification true -failovertype SELECT -failovermethod BASIC -failoverretry 3 -failoverdelay 5 -clbgoal LONG -rlbgoal NONE -tafpolicy BASIC -preferred S772 -available S771
	

ASM Probleme:
https://www.hhutzler.de/blog/crs-doesnt-start-due-to-crs-1714-error/
https://www.nazmulhuda.info/how-to-map-asm-disk-to-physical-device

## relink:

In der MOS Note mit Doc.ID 1536057.1 " Relink The Oracle Grid Infrastructure Standalone (Restart) Installation Oracle Grid Infrastructure RAC/Cluster Installation (11.2 to 19c)" stehen die Einzelheiten. Hier aber zusammengefasst:

Schritt 1:
Relocate ALLER HA-Services auf einen der beiden Knoten:

srvctl relocate service -db $INSTANZ -service ${INSTANZ}HA -oldinst ${INSTANZ}$1 -newinst ${INSTANZ}$2
Zum Relocaten der HA-Services gibt es ein Skript, das auf die RAC-Knoten übertragen und ausgeführt werden kann: relocate_services.sh 
Usage des Skriptes:

./relocate_services.sh (Alle HA-Services, die nicht bereits auf dem anderen RAC-Server laufen, werden verschoben)
Schritt 2:

Stoppen ALLER Instanzen auf dem Knoten, auf dem die HA-Services nicht laufen.
srvctl stop instance -db <DB> -instance <INSTANZ>

Zum Stoppen und Starten aller Instanzen kann das folgende Skript genutzt werden (vorher auf die RAC-Knoten übertragen): start_stop_instances_relink_homes.sh 
Usage des Skriptes:

./start_stop_instances_relink_homes.sh stop (zum Stoppen ALLER Instanzen auf dem Server) 
./start_stop_instances_relink_homes.sh start (zum Starten ALLER Instanzen auf dem Server) 
./start_stop_instances_relink_homes.sh relink (zum Relinken ALLER RDBMS Homes auf dem Server) 
Schritt 3:

Checken ob RAC feature enable ist (nur zur Überprüfung)

ar -t $ORACLE_HOME/rdbms/lib/libknlopt.a|grep kcsm.o

Schritt 4:

Die Clusterservices runter fahren und den Autostart deaktivieren:

**Als User "root"**

/oracle/grid/19XX00/bin/crsctl stop crs
/oracle/grid/19XX00/bin/crsctl disable crs

Schritt 5:

unlock der Grid Infrastructure Oracle Home

**Als User "root"**

/oracle/grid/19XX00/crs/install/rootcrs.sh -unlock

Schritt 6:

Upgrade / Update des OS auf dem ersten Knoten

Schritt 7:
Nach Rückmeldung, dass OS auf z.B. SLES 12 SP5 ist oder wenn ein zypper update gemacht wurde:

**Als User "root"**

oracleasm status

Schritt 8:

Relink der Grid Infrastructure Oracle Home

**Als oracle  mit ASM Environment**

$ORACLE_HOME/bin/relink

Schritt 9:

Prüfen, ob RAC feature enable ist

ar -t $ORACLE_HOME/rdbms/lib/libknlopt.a|grep kcsm.o


Schritt 10a:

**Als User "root"**

/oracle/grid/<19XXXX>/rdbms/install/rootadd_rdbms.sh
Schritt 11:

**Als User "root"**

crsctl enable crs
oder als oracle mit ASM Environment:

sudo crsctl enable crs
Schritt 12:

**Als User "oracle"**

Alle RDBMS HOMES  relinken:

Das jeweilige Environment der Datenbank setzen 

relink all
durchführen.

Dafür gibt es ein Skript, das auf den die RAC-Knoten übertragen werden kann: start_stop_all_instances_relink_homes.sh

Schritt 13:

**Als User "root"**

/oracle/grid/<19XXXX>/crs/install/rootcrs.sh -lock

Schritt 14:
Starten ALLER Instanzen auf dem gepatchten Knoten:

srvctl start instance -db <DB> -instance <INSTANZ>
Siehe Skripte unter Schritt 2:

./start_stop_instances_relink_homes.sh start
Schritt 15:
(optional falls Cloud Control Agent installiert ist):

$AGENT_HOME/bin/emctl start agent
Schritt 16:
Danach Schritt 1 bis 5 wiederholen!


Schritt 17:
OS Upgrade/Update auf dem verbleibenden Knoten


Schritt 18:
Wiederholen von Schritt 7 bis 15


Schritt 19:
Überprüfung der Knoten

Schritt 20:
Freigabe der Datenbanken!
Fertig


