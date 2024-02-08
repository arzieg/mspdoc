
#############################
Oracle Multitenant Workshop
#############################

 DB Passwort: welcome1
   
 In Multitenant: 
   - Anmeldung per SQL über ServiceNamen und nicht mehr per SID! Die SID wäre die CDB, daher immer den ServiceName verwenden. 
	   PDB per service:	
		tnsnames.ora 
		user/pwd@<host>:<port>/<pdbname>
	   PDB von CDB ansprechen	
		alter session set container=<pdb>;
		show pdbs (wenn man an der pdb angemeldet ist, bekommt man nur eine Zeile angezeigt. Ist man an der cdb angemeldet, sieht man mehrere Zeilen)
	   OS-Ebene
	    export ORACLE_PDB_SID=<pdb>
		export ORACLE_SID=<cdb>

		  
 
	  
	  
Start / Stop
=============
alter pluggable database <pdb>|ALL open [READ WRITE|READ ONLY]

alter pluggable database <pdb>|ALL [EXCEPT pdb1,pdb2] close [IMMEDITATE | ABORT];

alter pluggable database <pdb> save state;  -> beim nächsten Reboot wird die PDB wieder in dem Zustand versetzt (z.B. geöffnet)

alter pluggable database <pdb> discard state;  -> vergessen den gespreicherten Zustand


in der PDB:

    alter session set container=<pdb>

    startup [FORCE][RESTRICT][OPEN[READ WRITE| READ ONLY]]

    shutdown immediate

**Status der PDBs**

.. code-block:: bash

    show pdbs;

PDB Views

.. code-block:: bash

    col table_name format a30
    col comments format a50
    set linesize 90
    select * from dict where lower(comments) like '%pdb%';


B&R
====

* rman kann die cdb+alle pdbs auf einmal sichern 

rman - Helfer (für NON RAC Datenbanken):

    list failure   -> zeigt den Fehler an

    Prozess:
        list failure

        advise failure

        repair failure preview

        repair failure
        

Clone lokal in einer CDB
=========================

Wenn die PDB in NOARCHIVELOG Mode, dann muss die PDB in RO gesetzt werden, um Kopie konsistent zu erhalten. 
Wenn PDB in ARCHIVELOG MODE dann ist die Kopie in Konsistenz zum Zeitpunkt des Starts der Kopie

.. code-block:: bash

    create pluggable database <pdb> from <pdb_clone>
     file_name_convert = ('/<pdb_to_clone'/, '/pdb/');
    alter pluggable database <pdb> open;

Clone über zwei CDBs
=====================

.. code-block:: bash

    rem Erstelle Common User in Quell CDB
    create user c##clone_admin identified by welcome1 container=all;
    grant connect,resource to c##clone_admin CONTAINER=ALL;
    grant create pluggable database to c##clone_admin CONTAINER=ALL;
    grant sysoper to c##clone_admin CONTAINER=ALL;

    rem Erstelle DB Link von Ziel CDB nach Quell CDB
    DROP PUBLIC DATABASE LINK db_link_to_<tns_source>;
    CREATE PUBLIC DATABASE LINK db_link_to_<tns_source>
    CONNECT TO c##clone_admin IDENTIFIED BY welcome1 USING '<tns_source>';

    rem Clone von Quell CDB nach Ziel CDB
    create pluggable database <pdb> FROM <pdb_clone>@db_link_to_<tns_source>
     FILE_NAME_CONVERT = ('/<tns_source>/<pdb_clone>>',
                          '/<tns>/<pdb>') ;

Erstellen eines refreshable Clones
===================================

REFRESH MODE muss in dem Befehl CREATE PLUGGABLE DATABASE angegeben wird. Dabei stehen folgende REFRESH Möglichkeiten zur Auswahl:

* NONE
* MANUAL
* EVERY 5 MINUTES
  
Die Refreshable Clone PDB kann nur im Status MOUNTED aktualisiert werden. Im obigen Beispiel geschieht dieses alle 5 Minuten. Mit der Option "MANUAL" sind 
die Aktualisierung manuell durchzuführen.

.. code-block:: bash

    rem Erstelle Common User in Quell CDB
    create user c##clone_admin identified by welcome1 container=all;
    grant connect,resource to c##clone_admin CONTAINER=ALL;
    grant create pluggable database to c##clone_admin CONTAINER=ALL;
    grant sysoper to c##clone_admin CONTAINER=ALL;

    rem Erstelle DB Link von Ziel CDB nach Quell CDB
    DROP PUBLIC DATABASE LINK db_link_to_<tns_source>;
    CREATE PUBLIC DATABASE LINK db_link_to_<tns_source> 
    CONNECT TO c##clone_admin IDENTIFIED BY welcome1 USING '<tns_source>';

    rem Erstelle DB Link von Ziel CDB nach Quell CDB
    create pluggable database <pdb>> FROM <pdb_clone>@db_link_to_<tns_source>
     FILE_NAME_CONVERT = ('/<tns_source>/<pdb_clone>',
                          '/<tns>/<pdb>') 
     REFRESH MODE EVERY 5 MINUTES;

Ein automatisches Refresh, wie im obigen Beispiel, erfolgt nur im Status MOUNTED. Die Klon-PDB kann aber auch READ ONLY geöffnet werden. 
Damit stoppt aber auch die Aktualisierung! Dieses ist also ganz anders als bei Active Data Guard, wo ja auch die geöffnete Standby-Datenbank aktualisiert wird.

**Status der PDBs**

.. code-block:: bash

    set linesize 200
    col pdb_name format a20
    col refresh_mode format a12
    select d.dbid as CDB_ID,d.name as CDB_NAME,pdb_id,pdb_name,refresh_mode,refresh_interval,FOREIGN_CDB_DBID,FOREIGN_PDB_ID from v$database d,cdb_pdbs;


**manueller Refresh**

.. code-block:: bash

    alter session set container=<pdb>;
    alter pluggable database close immediate;
    alter pluggable database refresh;
    alter pluggable database open read only;


Data Dictionary
================

**Schichtenmodell - verschiedene Hierarchien**

CDB_xxx Alle Objekte in CDB und alle PDBs
  DBA_xxx alle Objekte in einer PDB
    ALL_xxx Alle Objekte die von einem DB User nutzbar sind
      USER_xxx Objekte des aktuellen DB-Users

``select view_name from dba_views where view_name like 'CDB%'``

In Oracle 12 noch die Vorstellung, das folgende Tablespaces auf CDB Ebene genutzt werden:
    SYSTEM
    
    SYSAUX
    
    UNDO 
    
    TEMP

Mittlerweile hast sich die Sicht geändert, UNDO und TEMP sollen zu den PDBs gehören.
shared UNDO = UNDO in der CDB. 
local UNDO = UNDO in der PDB (man kann Flashback machen)-
DBCA benutzt automatisch local UNDO.
Bei create database clause muss man LOCAL UNDO ON nutzen (default ist shared UNDO)!

User
=====
PDB Administration niemals mit SYS oder SYSTEM. Dies sind common user und überall verwendbar.
SYS und SYSTEM sind nur für den zentralen Betrieb gedacht.  Auf der pdb soll mit einem pdbadmin 
gearbeitet werden. 

.. code-block:: bash
       
    sqlplus system/<passwort>@$HOSTNAME:1521/<pdb>
    create user pdbadmin identified by <passwort>;
    grant dba to pdbadmin;   

C##<user> = common user auf der CDB Ebene; können dann per Role auch auf alle PDBs zugreifen. 
            Eher ein Namenskonvention. 
            Kann man per COMMON_USER_PREFIX ändern, aber sollte man vlt. nicht. 

Rechte für alle PDBs:
``grant create session to c##<name> container=all``

Rechte für einzelne PDBs:
```grant create session to c##name container=pdb1,pdb2;``

Ressourcen
============
Ressourcenpläne können auf Ebene der CDB und der PDB erstellt werden. 

Shares
-------
Man kan Shares definieren, d.h. x% einer CPU Leistung (was auch immer das ist)
Man definiert dann je PDB den Anteil an CPU Ressourcen: 
Beispiel: 

    pdb1  share=50%  (pdb hat doppelt soviel CPU Ressurcen ggü. pdb2)

    pdb2  share=25%

    pdb3  share=25%

Man kann auch über 100% vergeben, dann ist der einzelne Share "weniger" Wert. Anteil an den Ressourcen = share/summe(shares).
Die shares stellen MIN Werte da, d.h. wenn mehr freie Ressourcen zur Verfügung stehen, dann bekommt die PDB diese auch. 

Limits
-------
Limits stellen harte Begrenzungen zur Nutzung von Ressourcen dar. Damit ist es also möglich, dass Ressourcen ungenutzt bleiben, obwohl die angefordert werden. 
Daher ist mit Limits sehr vorsichtig umzugehen.


Erstellung ohne Profile
-------------------------
Ein Ressourcen Plan wird mit dem PL/SQL-Package DBMS_RESOURCE_MANAGER erstellt.

.. code-block:: bash

    begin
        DBMS_RESOURCE_MANAGER.CREATE_PENDING_AREA();
        DBMS_RESOURCE_MANAGER.CREATE_CDB_PLAN(
        plan => 'cdb_plan',
        comment => 'CDB Resource Plan');
        DBMS_RESOURCE_MANAGER.CREATE_CDB_PLAN_DIRECTIVE(
        plan => 'cdb_plan', 
        pluggable_database => '<pdb1>', 
        shares => 5, 
        utilization_limit => 25,
        parallel_server_limit => 20);
        DBMS_RESOURCE_MANAGER.CREATE_CDB_PLAN_DIRECTIVE(
        plan => 'cdb_plan', 
        pluggable_database => '<pdb2>', 
        shares => 10, 
        utilization_limit => 10,
        parallel_server_limit => 20);
        DBMS_RESOURCE_MANAGER.VALIDATE_PENDING_AREA();
        DBMS_RESOURCE_MANAGER.SUBMIT_PENDING_AREA();
    end;

Erstellung mit Profile
-----------------------
Man kann auch Resourcenpläne (templates) erstellen und diese dann einzelnen PDBs zuordnen. 

.. code-block:: bash

    begin
        DBMS_RESOURCE_MANAGER.CREATE_PENDING_AREA();
        DBMS_RESOURCE_MANAGER.CREATE_CDB_PLAN(
        plan => 'cdb_plan',
        comment => 'CDB Resource Plan');
        DBMS_RESOURCE_MANAGER.CREATE_CDB_PROFILE_DIRECTIVE(
        plan => 'cdb_plan', 
        profile => 'sla1', 
        shares => 10, 
        utilization_limit => 100,
        parallel_server_limit => 100);
        DBMS_RESOURCE_MANAGER.CREATE_CDB_PROFILE_DIRECTIVE(
        plan => 'cdb_plan', 
        profile => 'sla2', 
        shares => 5, 
        utilization_limit => 70,
        parallel_server_limit => 70);
        DBMS_RESOURCE_MANAGER.CREATE_CDB_PROFILE_DIRECTIVE(
        plan => 'cdb_plan', 
        profile => 'sla3', 
        shares => 2, 
        utilization_limit => 50,
        parallel_server_limit => 50);
        DBMS_RESOURCE_MANAGER.VALIDATE_PENDING_AREA();
        DBMS_RESOURCE_MANAGER.SUBMIT_PENDING_AREA();
    end;

    
        
    ALTER SESSION SET container=<pdb>;
    ALTER SYSTEM SET DB_PERFORMANCE_PROFILE=sla3 SCOPE=spfile;
    ALTER PLUGGABLE DATABASE CLOSE IMMEDIATE;
    ALTER PLUGGABLE DATABASE OPEN;
    show parameter DB_PERFORMANCE_PROFILE
    

Damit über ein alter system nicht selber das Profil geändert werden kann, kann man ein lockdown Profil erzeugen. Hier möchte man nur das DB_PERFORMANCE_PROFILE nicht
gesetzt werden kann.

.. code-block:: bash

    CREATE LOCKDOWN PROFILE rfix;
    ALTER LOCKDOWN PROFILE rfix DISABLE STATEMENT = ('ALTER SYSTEM') CLAUSE=('SET') OPTION=('DB_PERFORMANCE_PROFILE');
    col profile_name format a20
    col rule_type format a20
    col rule format a20
    col clause format a10
    col clause_option format a30
    col option_value format a20
    set linesize 400
    select profile_name,rule_type,rule,clause,clause_option,status from dba_lockdown_profiles;
    ALTER SESSION SET container=<pdb>;
    ALTER SYSTEM SET pdb_lockdown=rfix;

Anzeigen von CDB Ressourcenplänen
-----------------------------------

.. code-block:: bash

    COLUMN PLAN FORMAT A30
    COLUMN STATUS FORMAT A10
    COLUMN COMMENTS FORMAT A35
    SELECT PLAN, STATUS, COMMENTS FROM DBA_CDB_RSRC_PLANS ORDER BY PLAN;

    set linesize 200
    col PLAN HEADING 'Plan' FORMAT A24
    col PLUGGABLE_DATABASE HEADING 'PDB' FORMAT A25
    col PROFILE format a15

    col SHARES HEADING 'Shares' FORMAT 999
    col UTILIZATION_LIMIT HEADING 'Utilization|Limit' FORMAT 999
    col PARALLEL_SERVER_LIMIT HEADING 'Parallel|Server|Limit' FORMAT 999 
    SELECT PLAN, PLUGGABLE_DATABASE, PROFILE, SHARES, UTILIZATION_LIMIT, PARALLEL_SERVER_LIMIT FROM DBA_CDB_RSRC_PLAN_DIRECTIVES ORDER BY PLAN;

Löschen von Ressourceplänen
----------------------------

.. code-block:: bash

    begin
        DBMS_RESOURCE_MANAGER.CREATE_PENDING_AREA();
        DBMS_RESOURCE_MANAGER.DELETE_CDB_PLAN(plan => 'cdb_plan');
        DBMS_RESOURCE_MANAGER.VALIDATE_PENDING_AREA();
        DBMS_RESOURCE_MANAGER.SUBMIT_PENDING_AREA();
    end;
    /

Memory Management für CDB/PDB
------------------------------

AMM = automatisches Management

    MEMORY_TARGET

    * gibt den systemweiten nutzbaren Speicher an
    * eher für kleine Datenbanken sinnvoll

    Empfehlung bei großen Systemen: 

    * SGA_TARGET
    * PGA_AGGREGATE_TARGET


Maximum Availability Architektur
==================================

RTO := Zeit für die Wiederherstellung nach einem Ausfall. 
RPO := Bis zu welchem Zeitpunkt und Zustand kann ich wiederherstellen


Dataguard
==========
 
 * wenn auf der primären Seite eine PDB angelegt wird, wird diese auch auf der standby Seite angelegt.
 * Für produktive Umgebungen fehlt noch der TEMP Tablespace auf der Standby Seite, der vom TEMP Talbespace des primären Seite abweichen darf und daher manuell angelegt werden muss.
 * Switchover

.. code-block:: bash
    dgmgrl 
        connect sys/<passwort>@PDB-Prod
        switchover to PDB-Repli;
        show configuration verbose
    exit
    


CLONING mit /ohne Dataguard
=============================


Cold Clone (Source R/O) := Während der Clone Erstellung wird die Source DB in den real-only Zustand versetzt. DB Blöcke werden von Source DB auf Clone DB 1:1 kopiert. 
    Neue PDB als Clone der PDB$SEED werden automatisch auch auf Standby angelegt, da PDB$SEED (R/O) auch auf der Standby Seiten vorhanden. Es erfolgt keine Datenübertragung 
    auf die StandBy Seite, da PDB$SEED da ja bereits vorhanden. 

Hot Clone (Source R/W)  := 
    PDB Hot Cloning geht mit 19c nur zur Primary CDB. Standby DB kann anschließend bspw. mit RMAN oder Cloning angelegt werden. 
    Diese Einschränkung gilt, da 2 parallele Recovery Ströme mit 19c nicht mögliche sind (es läuft ja bereits auf CDB Ebene ein RecoveryStrom von Primary -> StandBy)

    Trick: lokaler R/W Cline enier "PDB ohne Standby" in eine "PDB mit Standby". 
      CDB DB link auf sich selbst anlegen, wird mit DG repliziert. Auf der Standby Seite wird der Link dann zu einem Link auf die Primary DB

Remote Clone (Read/Only): 
    1. DB User für Cloning auf Quell DB einrichten
    2. DB Linkzur Siurce auf Primary CDB anlegen. 
    3. DB Link zur Source auf Standy CDB als Parameter eintragen.
    4. Source DB auf read-only setzen
    5. Standby Absicherung mittels RMAN oder über weiteren Cold Clone der Primary ÜDB mit parallerler Absicherung auf Standby Seite. 

Remote Clone (Read/Write)
    tbd

In Oracle 21c ff. gibt es eine PDB Recovery Isolation Mode, da spart man sich die Workarrounds, um ein Hot Clone zu machen. 

In der Cloud gibt es noch ein **Refreshable PDB Switchover**. Hier erfolgt in einem zu definierenden Zyklus ein Fresch von PDB1 -> PDB1' 

PDB und RAC
============

Spielvariante: PDB wird nur auf einer Instanz gestartet, in zweiter Instanz im Status mounted. Bei Ausfall open der Instanz. 
                Wenn Auslastung zu hoch, weiterer RAC Server, dann verschieben auf einen anderen Server usw.

PDB Placement: wie kann ich steuern, wo die PDB läuft?
                ``srvctl add service -db CDB -service PDB1 -pdb PDB1 -prefered CDB1 -available CDB3,CDB2 -tafpolicy BASIC``

in 23c neues Policy Management für automatische PDB Ressourcen Verteilung

* die Clusterware erkenntn nun die betriebenen PDBs
* automatische Verteilung von PDBs "Floating PDBs"
* zusätzliche Cluster Attribute
    * cardinalität - maximal die der CDB
    * rank - je höher desto wichtiger
    * mincpuunit (minimal garantierte CPU 1/100)
    * maxcpu (maximales Limit PDB)

Weiterhin ist *administrator mangaged* weiterhin möglich.      
                 
PDB verschieben
===============
* Zeichensatz in PDB ist kompatibel zu Zeichensatz in Ziel-CDB (Oracle Empfehlung: AL32UTF8)
* Ziel CDB mit gleichen DB Optionen wie Source CDB
* Wenn common user in der pdb eingesetzt werden, dann müssen die auf beiden Seiten existieren. 
* typischerweise sollte auf beiden Seite der gleiche Patchlevel existieren. Es geht auch von alt nach neu, dann ein datapatch notwendig. 

Vorgehen
--------
* xml Datei enthält Daten der PDB Struktur
* pdb wird ausgehänt
* pdb und xml werden auf Ziel übertragen
* pdb Kompatibilität wird mit CDB geprüft
    view: *pdb_plug_in_violations* prüfen
* pdb wird eingehängt
    eigentlich ein create Befehl. Hier zwei Möglichkeiten: COPY und NOCOPY. Bei NOCOPY stehen die Datenfiles bereits dort, wo sie hingehören. Bei COPY wird das noch dahin 
    kopiert. Bei NOCOPY mit SOURCE_FILE_NAME_CONVERT wird source nach dest im xml-file umgesetzt. Bei COPY muss FILE_NAME_CONVERT angegeben.
    TEMPFILE REUSE notwendig, wenn man das Tempfile mit kopiert, ansonsten muss man es neu anlegen. 
* alte pdb wird gelöscht
    ``DROP PLUGGABLE DATABASE <PDB> INCLUDING DATAFILES``

Alternative zu unplug/plug (minimale Downtime)
------------------------------------------------
*  common user auf source db erstellen
*  dblink von Ziel-CDB auf Source-CDB erstellen
*  PDB von Ziel CDB über DBLink clonen
     create pluggable database <pdb> from pdb_to_relocate@dblink file_name_convert = ('/$tns_source/','/$tns_dest/') relocate availability [NORMAL|MAX];
     *relocate availability* bei einer Migration von einer PDB ok, wenn PDB in einer CDB mit mehreren PDBs nicht so sinnvoll. 
*  PDB Clone konsistent machen
*  PDB Clone aktualisieren bis ...
*  ... PDB Clone aktivieren  (``alter pluggable database <pdbtorelocat> open;``)

Wenn eine PDB von einem Server auf einen anderen HOST wechselt, ändert sich der HOST in der TNSAlias. Ziel stabiler TNS Alias. Lösung: remote Listener, 
CDB registrieren als Service bei allen Remote Listenern
Durch PDB Verschiebung wird DB Sitzung abgebrochen (behalten aber gleichen TNS Alias). Mit *Application Continuity* werden offene Transaktionen zwischengespeichert. Hier
RAC oder Active Data Guard Lizenz notwendig. 


Patchen
=========

Update
-------

1. Unplug der PDB

.. code-block:: bash

    
    # Unplug der PDB aus Source
    select file_name from dba_data_files;
    alter pluggable database <pdb> close immediate;
    alter pluggable database <pdb> unplug into '/tmp/<pdb>.xml';
    drop pluggable database <pdb>;
    
    cat /tmp/$pdb_to_patch.xml

2. copy aller Dateien zur neuen CDB
3. Kompatibilitätsprüfung
   
.. code-block:: bash

    # Kompatibilitätsprüfung für Plugin
    export ORACLE_HOME=$ORACLE_HOME_MP
    export ORACLE_SID=$ORACLE_SID_MP
    SET SERVEROUTPUT ON
    var compatible varchar2(300);
    BEGIN
        :compatible := 
        CASE DBMS_PDB.CHECK_PLUG_COMPATIBILITY(
         pdb_descr_file => '/tmp/<pdb>.xml',
         pdb_name => '<pdb>')
        WHEN TRUE THEN 'YES, the PDB is compatible. You can go on'
                ELSE 'NO, PDB is not compatible and cannot be plugged in!'
        END;
    END;
    /
    exec DBMS_OUTPUT.PUT_LINE(:compatible);
    exit

Ergebnis sollte "NO, PDB is not compatible ..." sein.

4. Status anschauen
   
.. code-block:: bash

    COL time FORMAT a10
    COL name FORMAT a15
    COL cause FORMAT a10 WRAP
    COL type FORMAT a8
    COL message FORMAT a40 WRAP
    COL action FORMAT a40 WRAP
    COL con_id FORMAT 9999
    COL line FORMAT 9999
    COL error_number FORMAT 9999
    SET LINESIZE 200 
    SET PAGESIZE 1000
    SELECT name,type,cause,message FROM pdb_plug_in_violations WHERE status='PENDING';

Sollten nur Fehler auftreten, dass die Patchstände nicht i.O. sind. 

5. PDB registrieren in der neuen CDB

.. code-block:: bash

    # Plug In
    connect / as sysdba
    CREATE PLUGGABLE DATABASE <pdb> USING '/tmp/<pdb>'
        NOCOPY
        SOURCE_FILE_NAME_CONVERT = ('/<source_sid>/', '/<dest_sid>/')
        TEMPFILE REUSE; 
    alter pluggable database <pdb> open;
    show pdbs
    exit

show pdb zeigt die Datenbank im RESTRICTED Mode. 

6. datapatch und danach status der pdb

.. code-block:: bash

    cd $ORACLE_HOME/OPatch
    ./datapatch -verbose

    sql / as sysdba
    show pdbs

Datenbank weiterhin in RESTRICTED Mode

7. PDB stoppen/starten und utlrp durchführen

.. code-block:: bash

    alter pluggable database $pdb_to_patch close immediate;
    alter pluggable database $pdb_to_patch open;
    show pdbs

    connect sys/<pwd>@<HOSTNAME>:1521/<pdb> as sysdba
    @?/rdbms/admin/utlrp.sql


Fallback/Downgrade
-------------------
PDB wieder auf die ungepatchte CDB verschieben und ein downgrade durchführen

1. Kopieren der SQLpatch-Dateien vom gepatchten Oracle Home in das ungepatchte Oracle Home
   Die Utility "datapatch" muß wissen, welche Schritte des Patch-Rollbacks in der PDB durchgeführt werden sollen. Dazu müssen die entsprechenden SQLPatch-Dateien 
   in das ungepatchte Oracle Home kopiert werden (i.d.R. sind es ja mehrere DB Patch, JAVA Patch, OPatch ...).

   .. code-block:: bash
   
    cp -R $ORACLE_HOME_PATCHED/sqlpatch/<PATCHNR> $ORACLE_HOME_OP/sqlpatch/.
    
2. unplug

.. code-block:: bash

    connect / as sysdba
    select file_name from dba_data_files;
    alter pluggable database <pdb> close immediate;
    alter pluggable database <pdb> unplug into '/tmp/<pdb>.xml';
    drop pluggable database <pdb>;
    exit
    
    cat /tmp/<pdb>.xml

3. Kopieren in das Zielverzeichnis

.. code-block:: bash

4. Kompatibilitätsprüfung

.. code-block:: bash
   
    connect / as sysdba
    SET SERVEROUTPUT ON
    var compatible varchar2(300);
    BEGIN
        :compatible := 
        CASE DBMS_PDB.CHECK_PLUG_COMPATIBILITY(
        pdb_descr_file => '/tmp/$pdb_to_patch.xml',
        pdb_name => '$pdb_to_patch')
        WHEN TRUE THEN 'YES, the PDB is compatible. You can go on'
                ELSE 'NO, PDB is not compatible and cannot be plugged in!'
        END;
    END;
    /
    exec DBMS_OUTPUT.PUT_LINE(:compatible);
    exit

Ergebnis: PDB ist nicht kompatibel

5. Grund anschauen 

.. code-block:: bash

    connect / as sysdba
    COL time FORMAT a10
    COL name FORMAT a15
    COL cause FORMAT a10 WRAP
    COL type FORMAT a8
    COL message FORMAT a40 WRAP
    COL action FORMAT a40 WRAP
    COL con_id FORMAT 9999
    COL line FORMAT 9999
    COL error_number FORMAT 9999
    SET LINESIZE 200 
    SET PAGESIZE 1000
    SELECT name,type,cause,message FROM pdb_plug_in_violations WHERE status='PENDING';

Ergebnis: CDB Version stimmt nicht mit PDB Version (höher) überein


6. PDB registrieren

.. code-block:: bash

    connect / as sysdba
    CREATE PLUGGABLE DATABASE <pdb> USING '/tmp/<pdb>.xml'
    NOCOPY
    SOURCE_FILE_NAME_CONVERT = ('/source_sid/', '/dest_sid/')
    TEMPFILE REUSE; 
    alter pluggable database <pdb> open;
    show pdbs
    exit
    
Ergebnis: PDB ist registriert, aber im RESTRICTED Mode

7. datapatch durchführen

.. code-block:: bash

    cd $ORACLE_HOME/OPatch
    ./datapatch -verbose

8. utlrp.sql ausführen

.. code-block:: bash

    $ORACLE_HOME/bin/sqlplus /nolog << EOI
    connect sys/<pwd>@HOSTNAME:1521/<pdb> as sysdba
    @?/rdbms/admin/utlrp.sql
    exit
    
9. PDB durchstarten

.. code-block:: bash
    
    connect / as sysdba
    alter pluggable database <pdb> close immediate;
    alter pluggable database <pdb> open;
    show pdbs
    

Migration von NonCDB to CDB
=============================

1. Erstellen einer nonCDB
   
   .. code-block:: bash

        dbca -silent -createDatabase \
        -templateName General_Purpose.dbc \
        -gdbname noncdb1 -sid noncdb1 -responseFile NO_VALUE \
        -characterSet AL32UTF8 \
        -sysPassword welcome1 \
        -systemPassword welcome1 \
        -createAsContainerDatabase false \
        -databaseType MULTIPURPOSE \
        -automaticMemoryManagement false \
        -totalMemory 2048 \
        -storageType FS \
        -datafileDestination "$ORADATA_DIR" \
        -redoLogFileSize 50 \
        -emConfiguration NONE \
        -ignorePreReqs

2. Erzeugen der XML-Datei für Kompatibilitätscheck
   
    .. code-block:: bash
       
        export ORACLE_SID=noncdb1

        connect / as sysdba
        select file_name from dba_data_files;
        shutdown immediate
        startup mount
        alter database open read only;
        begin
        dbms_pdb.describe(pdb_descr_file => '/tmp/noncdb1.xml');
        end;
        /
        
        shutdown immediate
        
        cat /tmp/noncdb1.xml
        
3. Vorbereiten der CDB
   
   .. code-block:: bash
   
        export new_pdb_name=<pdb>
        mkdir -p /u01/app/oracle/oradata/$ORACLE_SID_CDB/<pdb>

        scp /tmp/<pdb>.xml newCDB-host:/tmp

4. Kompatibilitäts check gegen neue CDB
   
   .. code-block:: bash
    
        export new_pdb_name=<pdb>
        export ORACLE_SID=<orasid-cdb>
        
        connect sys/<pwd>@<cdb> as sysdba
        SET SERVEROUTPUT ON
        var compatible varchar2(300);
        BEGIN
        :compatible := 
        CASE DBMS_PDB.CHECK_PLUG_COMPATIBILITY(
        pdb_descr_file => '/tmp/noncdb1.xml',
        pdb_name => '<pdb>>')
        WHEN TRUE THEN 'YES, the PDB is compatible. You can go on'
                ELSE 'NO, PDB is not compatible and cannot be plugged in!'
        END;
        END;
        /
        exec DBMS_OUTPUT.PUT_LINE(:compatible);
        
5. Report anzeigen

    .. code-block:: bash

        COL time FORMAT a10
        COL name FORMAT a15
        COL cause FORMAT a10 WRAP
        COL type FORMAT a8
        COL message FORMAT a40 WRAP
        COL action FORMAT a40 WRAP
        COL con_id FORMAT 9999
        COL line FORMAT 9999
        COL error_number FORMAT 9999
        SET LINESIZE 200
        SELECT name,type,cause,message FROM pdb_plug_in_violations WHERE status='PENDING';

6. registrieren der non-cdb kopie als pdb

    .. code-block:: bash

        connect / as sysdba
        CREATE PLUGGABLE DATABASE <pdb> USING '/tmp/noncdb1.xml'
        COPY
        FILE_NAME_CONVERT = ('/NONCDB1/', '/$ORACLE_SID_CDB/<pdb>/')
        USER_TABLESPACES=('users'); 
        

7. Prüfung

    .. code-block:: bash

        
        connect / as sysdba
        col name format a20
        select v.con_id,v.name,v.open_mode,v.restricted,c.status
        from v\$pdbs v,cdb_pdbs c
        where v.con_id=c.pdb_id;
        
PDB hat den Status NEW

8. PDB-spezifische Inhalte in der neuen PDB erstellen
   Bevor die neue PDB geöffnet wird (und das ist ganz wichtig), muß die neue PDB erst die internen Strukturen einer PDB bekommen. Dazu wird das Skript noncdb_to_pdb.sql aus dem Verzeichnis $ORACLE_HOME/rdbms/admin gestartet
   
   .. code-block:: bash

        connect / as sysdba
        alter session set container=<pdb>;
        @?/rdbms/admin/noncdb_to_pdb.sql
        exit 
        
9.  PDB öffnen

    .. code-block:: bash

        sqlplus /nolog << EOI
        connect / as sysdba
        ALTER PLUGGABLE DATABASE $new_pdb_name open;
        exit
        EOI



















ToDo: 
=====
* mit Oracle 23 ist nur CDB/PDB erlaubt. Wie erfolgt der Update auf Oracle 19/NonCDB -> 23/CDB? Erst Wechsel von NonCDB->CDB und dann nach Oracle23 oder ist das ein Schritt? 
* Anpassung Oracle emcadm2 notwendig? 
* Usermanagement in c##
* 



