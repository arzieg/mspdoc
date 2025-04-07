# Oracle Allg. Administration

## Anmelden PDB/CDB
	ssh icrac
	su - oracle
	environment auswählen
	sql
		show pdbs
		alter session set container="<container>"
		

## Parameter

`alter system reset open_cursors scope=spfile sid='*';`    reset eines Parametes

### pfile erzeugen

`create pfile='initN221.ora' from spfile;`


### Controlfiles

#### ohne asm

1. Infos
```
SELECT name FROM v$controlfile;
show parameter control_files;
```
2. alle controlfiles angeben

```
ALTER SYSTEM SET CONTROL_FILES='/data/app/oracle/oradata/PRIM/control01.ctl','/data/app/oracle/oradata/PRIM/control02.ctl','/data/app/oracle/oradata/PRIM/control03.ctl' SCOPE=spfile;
```

3. shutdown immediate

```
sqlplus '/ as sysdba'
SQL> shutdown immediate;
SQL> startup nomount;
```

4. Copy the controlfile to it's two new locations

```
RMAN> connect target /
connected to target database: N22 (not mounted)
RMAN> restore controlfile from '+N22_DATA/N22/CONTROLFILE/current.261.1177579279';
Starting restore at 21.08.2024 11:22:32
using target database control file instead of recovery catalog
allocated channel: ORA_DISK_1
channel ORA_DISK_1: SID=9 instance=n221 device type=DISK
channel ORA_DISK_1: copied control file copy                                                                                                                                                output file name=+N22_DATA/N22/CONTROLFILE/current.261.1177579279
output file name=+N22_MLOG/N22/CONTROLFILE/current.256.1177586553
output file name=+N22_OLOG/N22/CONTROLFILE/current.256.1177586555
Finished restore at 21.08.2024 11:22:40
```

5.  Modify the spfile with the name of new control file
`alter system set control_files='+N22_DATA/N22/CONTROLFILE/current.261.1177579279','+N22_MLOG/N22/CONTROLFILE/current.256.1177586553','+N22_OLOG/N22/CONTROLFILE/current.256.1177586555' scope=spfile sid='*';`

6. Shutdown / Startup the database and check the new control file

```
shutdown immediate
startup 
SQL> select name from v$controlfile;

NAME
--------------------------------------------------------------------------------
+N22_DATA/N22/CONTROLFILE/current.261.1177579279
+N22_MLOG/N22/CONTROLFILE/current.256.1177586553
+N22_OLOG/N22/CONTROLFILE/current.256.1177586555
```


#### mit asm

MOS Doc ID 2006213.1

1. Infos
SELECT name FROM v$controlfile;
show parameter control_files;

2. alle controlfiles angeben
alter system set control_files='+N22_DATA/N22/CONTROLFILE/current.261.1177579279','+N22_MLOG','+N22_OLOG' scope=spfile sid='*';




## Temporäre Tablespaces:

```
column file_name format a30
select file_name, tablespace_name, autoextensible from dba_temp_files;
alter database tempfile '/oracle/OLI/sapdata1/OLI/datafile/o1_mf_temp_j2tkytr5_.tmp' autoextend on;
```

## Schema löschen: 

1. Schema Dumpen (zur Sicherheit)
   Hierzu muss der User der den Export durchführt Berechtigung haben dort reinzuschreiben. 
   Welche Directories sind ggfs. schon definiert:
	`SELECT * FROM ALL_DIRECTORIES`
		
		Beispiel, wenn man das spezifisch einem User zuordnen will. 
		CONN / AS SYSDBA
		ALTER USER scott IDENTIFIED BY tiger ACCOUNT UNLOCK;
		CREATE OR REPLACE DIRECTORY test_dir AS '/u01/app/oracle/oradata/';
		GRANT READ, WRITE ON DIRECTORY test_dir TO scott;
		
		create or replace directory DATAPUMP_DIR as '/oracle/ASST/admin/ASST/dpdump/';
		

   Export starten, Username SYSTEM, Passwort: ... 

   `expdp schemas=JUTHIE directory=DATAPUMP_DIR dumpfile=JUTHIE_ZAPPL2T.dmp logfile=expdpJUTHIE.log content=ALL cluster=N version=COMPATIBLE`

2. User Locken und warten

	`select USERNAME,ACCOUNT_STATUS,LOCK_DATE from dba_users where username='JUTHIE';`
	`alter user JUTHIE account lock;`
	
3. User dropen cascade

	`drop user <USER> cascade`

4. Ist der Tablespace leer? 
    ```
    SELECT UT.TABLESPACE_NAME "TABLESPACE", COUNT (US.SEGMENT_NAME) "NUM SEGMENTS"
    FROM DBA_TABLESPACES UT, DBA_SEGMENTS US
    WHERE UT.TABLESPACE_NAME = US.TABLESPACE_NAME
    GROUP BY (UT.TABLESPACE_NAME)
    ORDER BY COUNT (US.SEGMENT_NAME) DESC;
    ```

wenn ja: 
	 
     `drop tablespace DRUCKOUTPUT including contents and datafiles;`


## Oracle Debugen

```
sqlplus / as sysdba
oradebug setospid <pid> <<<<<<<<< where this is the oracle pid identified in the error file 
(you should grep for this first to ensure it is the spid )

oradebug unlimit
oradebug dump errorstack level 3
oradebug dump heapdump level 29
<leave 2 mins>
repeat
<leave 2 mins>
repeat
```

This assumes that any memory growth is in an individual process rather than in the SGA. 
You can also use OS tools to identify whether any process is growing unexpectedly.


## Logfiles:

```
column group# format 999999
column member format a80
SELECT a.group#, a.member, b.bytes FROM v$logfile a, v$log b WHERE a.group# = b.group#;
```



## WALLET

https://wiki.ez.edeka.net/pages/viewpage.action?pageId=84859350#RACContainerDatenbankinstallieren(Oracle19c)-Walletkonfiguration
https://www.oracle.com/webfolder/technetwork/de/community/dbadmin/tipps/eps/index.html
!! sqlnet.ora anpassen, sonst klappt test nicht !! 

```
# Als User "oracle":
mkdir -p $ORACLE_HOME/network/wallet
# Dann Wallet auf 1. Knoten anlegen
mkstore -wrl $ORACLE_HOME/network/wallet -create
 # Mit „mkstore -wrl <wallet_location> -createCredential <TNS-Alias> <user_name> <password>“ wird der User angelegt.
# Nun die Login Daten in dem Wallet speichern für den Zugriff auf die RMAN-Catalog Datenbank:
mkstore -wrl $ORACLE_HOME/network/wallet -createCredential rmanrep rman
 # Testen des Wallets:
sqlplus /@rmanrep
rman target '"/ as sysbackup"' catalog /@rmanrep
rman target / catalog /@rmanrep
# Kopieren auf den anderen knoten nach erfolgreichen Tests:
cd $ORACLE_HOME/network
ssh <2.racnode> mkdir -p $ORACLE_HOME/network/wallet
scp wallet/* <2.racnode>:$ORACLE_HOME/network/wallet

User anzeigen:
mkstore -wrl $ORACLE_HOME/network/wallet -listCredential
```


	
## Schema EX-/IMPORT 
(https://wiki.ez.edeka.net/pages/viewpage.action?pageId=18154047)

### NLS-Parameter

```
spool $ORACLE_HOME/dbs/Output_NLSpar.out
select * from nls_database_parameters;
spool off
```

### Abfrage der TBS-Größen und Anzahl der Segmente

```
spool $ORACLE_HOME/dbs/Output_TBS_seg.out
select decode(grouping(tablespace_name),1,'*SUM*',tablespace_name) tablespace_name,
decode(grouping(owner),1,'*SUM*',owner) owner,
decode(grouping(segment_type),1,'*SUM*',segment_type) segment_type,
trunc(sum(bytes/1024/1024)) MB, count(*)
from dba_segments
where owner in ('%') -- Hier ist der SCHEMANAME anzupassen
group by rollup(tablespace_name,owner,segment_type);
spool off
```

### Free Space in Tablespace

```sql
column "Tablespace" format a13
column "Used MB"    format 99,999,999
column "Free MB"    format 99,999,999
column "Total MB"   format 99,999,999
select
   fs.tablespace_name                          "Tablespace",
   (df.totalspace - fs.freespace)              "Used MB",
   fs.freespace                                "Free MB",
   df.totalspace                               "Total MB",
   round(100 * (fs.freespace / df.totalspace)) "Pct. Free"
from
   (select
      tablespace_name,
      round(sum(bytes) / 1048576) TotalSpace
   from
      dba_data_files
   group by
      tablespace_name
   ) df,
   (select
      tablespace_name,
      round(sum(bytes) / 1048576) FreeSpace
   from
      dba_free_space
   group by
      tablespace_name
   ) fs
where
   df.tablespace_name = fs.tablespace_name;
```

### Abfrage der default Tablespaceeinstellung

```
spool $ORACLE_HOME/dbs/Output_defaultTBS.out
select USERNAME,DEFAULT_TABLESPACE,TEMPORARY_TABLESPACE
from dba_users
where username in ('HADOOP_HIVE_SAND'); -- Hier ist der SCHEMANAME anzupassen
spool off
```

### Ausgabe des Patchstandes der Datenbank auf dem OS

`$ORACLE_HOME/OPatch/opatch lsinventory -invptrloc $ORACLE_HOME/oraInst.loc > $ORACLE_HOME/dbs/Output_OPATCHlsinv.out`

### Export mit Daten

	Abfrage SCN
	------------
	column current_scn format 9999999999999999
	select current_scn from v$database 

```
Parameterfile erstellen:
SCHEMAS=<schema1>[,<schema2>[,...]]
DIRECTORY=DATAPUMP_DIR
DUMPFILE=expdp_<schema>_<datum>_a%U.dmp[,expdp_<schema>_<datum>_b%U.dmp[,...]]
LOGFILE=expdp_<schema>_<datum>.log
PARALLEL=4
FILESIZE=30GB
CONTENT=ALL
CLUSTER=N
EXCLUDE=STATISTICS
ESTIMATE=STATISTICS
VERSION=COMPATIBLE
FLASHBACK_SCN=<current_scn der DB>
```

und dann: `expdp system@<DBService> parfile=/mnt/dump/expdp.par`

Data-Pump-Dir: SELECT owner, directory_name, directory_path FROM dba_directories WHERE directory_name='DATA_PUMP_DIR';
wenn user darauf zugreifen soll:  GRANT READ, WRITE ON DIRECTORY HADOOP_DUMP_DIR to HADOOP_HUE_ENTW; später dann REVOKE READ, WRITE ON DIRECTORY HADOOP_DUMP_DIR FROM HADOOP_HUE_ENTW;

Data-Pump-Jobs: `select * from dba_datapump_jobs;`

Status of DP Jobs:  `SELECT owner_name, job_name, operation, job_mode, state FROM dba_datapump_jobs where state='EXECUTING';`
					`SELECT owner_name, job_name, operation, job_mode, state, attached_sessions FROM dba_datapump_jobs;`

Waiting of some Resources? 

		col sql_text for a30
		col error_msg for a30
		select user_id, session_id, status, start_time, suspend_time, sql_text, error_number, error_msg from dba_resumable;

Wait events? `SELECT w.sid, w.event, w.seconds_in_wait FROM V$SESSION s, DBA_DATAPUMP_SESSIONS d, V$SESSION_WAIT w WHERE s.saddr = d.saddr AND s.sid = w.sid;`

Wenn im Alterfile "PMON Process Repeats ORA-4031 After Expdp Session Fails With ORA-4031 ("streams pool","unknown object","streams pool","fixed allocation callback") " erscheint, dann https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=43795449833762&id=2333337.1&_afrWindowMode=0&_adf.ctrl-state=n0n0kbwp5_4



### Import: 

Parameterfile
```
cat impdp_HADOOP_HIVE_SAND.par

	directory=DATA_PUMP_DIR
	dumpfile=HADOOP_HIVE_SAND.dmp
	logfile=IMPDP_HADOOP_HIVE_SAND_20210302.log
	parallel=1
	FULL=YES
	LOGTIME=ALL
	JOB_NAME=HADOOP_HIVE_SAND_imp
	remap_tablespace=HADOOP_DATA:HADOOP_HUE

impdp system@<DBService>  parfile=`pwd`/impdp_HADOOP_HIVE_SAND.par
```




## Oracle-Inventory

Installationshome steht in einer inventory.xml. Notfalls muss das manuell bearbeitet werden. 
	vi /oracle/oraInventory/ContentsXML/inventory.xml
	Beispiel: Oracle Cloneprozess neu starten. Dann muss in der inventory das Home-Verzeichnis gelöscht werden. 
	Besser: How To Clean Up The Inventory After Deleting The Oracle Home Manually Using OS Commands("rm-rf " or other)? (Doc ID 435219.1)
	https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=442225684451416&parent=EXTERNAL_SEARCH&sourceId=HOWTO&id=435219.1&_afrWindowMode=0&_adf.ctrl-state=10i4m4supz_260
	
	
	
## Usermanagement

```
CREATE USER TECH_VM_SECSCAN_DBL_LABOR
IDENTIFIED BY "Edeka123!!Edeka123!!Edeka123!!"
DEFAULT TABLESPACE SYSAUX
TEMPORARY TABLESPACE PSAPTEMP
PROFILE DEFAULT
ACCOUNT UNLOCK;

GRANT CONNECT TO tech_vm_secscan_dbl_labor;
GRANT DBA TO tech_vm_secscan_dbl_labor;
```

Passworthash: http://www.peasland.net/2016/02/18/oracle-12c-identified-by-values/
Vorgehen: 

```
show long
set long 10000000
SQL> select dbms_metadata.get_ddl('USER','TECH_VM_SECSCAN_DBL_LABOR') from dual;

DBMS_METADATA.GET_DDL('USER','TECH_VM_SECSCAN_DBL_LABOR')
--------------------------------------------------------------------------------

   CREATE USER "TECH_VM_SECSCAN_DBL_LABOR" IDENTIFIED BY VALUES 'S:<HASH ....>'
      DEFAULT TABLESPACE "SYSAUX"
      TEMPORARY TABLESPACE "PSAPTEMP"
 ALTER USER "TECH_VM_SECSCAN_DBL_LABOR" LOCAL TEMPORARY

DBMS_METADATA.GET_DDL('USER','TECH_VM_SECSCAN_DBL_LABOR')
--------------------------------------------------------------------------------
 TABLESPACE "PSAPTEMP"
```

## Memory:

Show SGA Usage

```
 SELECT ROUND (used.bytes / 1024 / 1024, 2) used_mb,
       ROUND (free.bytes / 1024 / 1024, 2) free_mb,
       ROUND (tot.bytes / 1024 / 1024, 2)  total_mb
  FROM (SELECT SUM (bytes) bytes
          FROM v$sgastat
         WHERE name != 'free memory') used,
       (SELECT SUM (bytes) bytes
          FROM v$sgastat
         WHERE name = 'free memory') free,
       (SELECT SUM (bytes) bytes FROM v$sgastat) tot;
```
	   
https://dbsguru.com/step-by-step-how-to-increase-sga-size-in-oracle/

## AHF

```
tfactl stopahf                  // Oracle ahf stoppen
neu: ahfctl stopahf
```
