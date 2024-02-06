
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

Man kan Shares definieren, d.h. x% einer CPU Leistung (was auch immer das ist)
Man definiert dann je PDB den Anteil an CPU Ressourcen: 
Beispiel: 

    pdb1  share=50%  (pdb hat doppelt soviel CPU Ressurcen ggü. pdb2)

    pdb2  share=25%

    pdb3  share=25%


Memory Management für CDB/PDB
------------------------------

AMM = automatisches Management

    MEMORY_TARGET

    * gibt den systemweiten nutzbaren Speicher an
    * eher für kleine Datenbanken sinnvoll

    Empfehlung bei großen Systemen: 

    * SGA_TARGET
    * PGA_AGGREGATE_TARGET







