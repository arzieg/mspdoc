.. _pacemaker:

##########
Pacemaker 
##########


Betriebsprozesse
*****************

Manueller Switch via crm Befehle
=================================
1. IPs offline setzen auf der primären Seite

.. code-block:: shell

    crm resource stop rsc_SAPHana_IP_<SID>_HDB10_01
    crm resource stop rsc_SAPHana_IP_<SID>_HDB10_02


2. Knoten stoppen, Reihenfolge standby -> worker -> master

.. code-block:: bash
    
    crm node standby <standby>
    crm node standby <worker>
    crm node standby <master>


3. IPs online nehmen (gehen automatisch beim Master online)

.. code-block:: bash

    crm resource start rsc_SAPHana_IP_<SID>_HDB10_01
    crm resource start rsc_SAPHana_IP_<SID>_HDB10_02


4. Knoten wieder online nehmen, Reihenfolge master -> worker -> standby

.. code-block:: bash

    crm node online <master>
    crm node online <worker>
    crm node online <standby>

Cluster in den Maintenancemode setzen
======================================
.. index:: maintenance

::
    
    crm configure property maintenance-mode=true
    crm configure property maintenance-mode=false
    crm configure show


Einzelne Ressourcen managen
============================
Ressourcen haben im SAPHana den Präfix rsc_<name>.

::

    # Ressource nicht managen
    crm resource unmanage <name>
    # Ressource wieder managen
    crm resource manage <name>
    # Ressource clearen
    crm_resource -r SAPHana_IP_<SID>_HDB10_01 -C
    # Ressourcen stoppen
    crm resource stop <name>
    # Stoppen der Ressource auf dem Knoten erzwingen
    crm_resource -r rsc_SAPHana_IP_<SID>_HDB10_01 --force-stop 
    # Ressource starten
    crm resource start <name>
    # Start der Ressource auf dem Knoten erzwingen
    crm_resource -r rsc_SAPHana_IP_<SID>_HDB10_01 --force-start
    # Ressourcen "verschieben"
    crm resource move <name> <node>
    # Alle Ressource-Fehler löschen
    crm_resource --cleanup
    # Löschen der Ressource - Fehler und reload 
    crm_resource -P


SAPHanaSR zeigt nur ein DC an, srHook wird nicht angezeigt
==============================================================
.. index:: srHook, crm_attribute

Wenn man sicher ist, wie der Zustand des Clusters ist, kann man das auch manuell setzen:
::
    
    # Datacenter manuell setzen
    crm_attribute -n hana_<sid>_glob_sec -v DC2 -t crm_config -s SAPHanaSR   # Failover DC
    crm_attribute -n hana_<sid>_glob_prim -v DC1 -t crm_config -s SAPHanaSE  # Primary DC

    # srHook manuell setzen
    crm_attribute -n hana_ysid>_glob_srHook -v SOK -t crm_config -s SAPHanaSR



Support
********
.. index:: hb_report

hb_report ausführen: 
::

    hb_report -u root -f "2020/08/10 11:00" -t "2020/08/11 11:00" /tmp/hb_report_log


Konfiguration
***************

SBD
=====

Stonith-Device: 
----------------

Die SBD Disks stehen in /etc/sysconfig/sbd

:: 
 
 for i in `egrep ^SBD_DEVICE /etc/sysconfig/sbd |cut -d '"' -f 2| tr ";" "\n"`; do sbd -d $i dump; done
 
 	==Dumping header on disk /dev/disk/by-id/scsi-<id>
	Header version     : 2.1
	UUID               : 132a8cfc-6153-4ceb-bb91-d01f42ed0825
	Number of slots    : 255
	Sector size        : 512
	Timeout (watchdog) : 30   <- watchdog * 2 = msgwait (passt hier nicht)
	Timeout (allocate) : 2
	Timeout (loop)     : 5
	Timeout (msgwait)  : 90   <- uups
	==Header on disk /dev/disk/by-id/scsi-<id> is dumped