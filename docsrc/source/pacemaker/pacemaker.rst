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
      Es werden dann Constraints gesetzt in der Konfiguration! Diese wird man mit
      crm resource clear <resourcenid> wieder los. Dies ist wichtig, da ansonsten der 
      Cluster beim Ausfall sich vielleicht anders verhält als erwartet.
    # Alle Ressource-Fehler löschen
    crm_resource --cleanup
    # Löschen der Ressource - Fehler und reload 
    crm_resource -P
    # Resource in Maintenance setzen
    crm resource maintenance msl_SAPHana_SR_<SID>_HDB10 on
    crm resource maintenance msl_SAPHana_SR_<SID>_HDB10 off
    # Wait for idle
    cs_wait_for_idle -n \<sec\>

Simulation
============
    crm_simulate -sL
    crm_simulate
    crm_simulate --simulate --xml-file <engine inpurt, f.i. /var/lib/pacemaker/cib/cib.xml>
    crm_simulate --simulate --live-check
    crm_simulate --simulate --live-check -VVVV -> Verbose 
    crm_simulate --simulate --live-check --show-scores
    
    # grafische Ausgabe
    crm_simulate --simulate --xml-file $FILENAME --dot-file $FILENAME.dot
        dot $FILENAME.dot -Tsvg > $FILENAME.svg

Logs
====
/var/lib/pacemaker
  pengine   - hier stehen CIB Dateien und deren Status in gepackter Form
  cib       - hier steht die Historie der cib-xml files


corosync-quorumtool
====================
Display the current state of quorum in the cluster and set vote quorum options.

Anzeige der definierten Knoten im Cluster und wie viele davon mindestens benötigt werden, damit der Cluster eine Aktion ausführen kann. 
Wenn das Quorum unterschritten wird, führt der Cluster keine Aktionen mehr aus (Sicherheit vor Datenverlust durch Aktionen auf einer defekten Seite).
Im Fall, wenn man einen 7 Knoten HANA Cluster (3+3+1) verwendet, dann liegt das Quorum bei 4. Wird je DC Seite ein StandBy rausgenommen, gehen dann 
durch einen DC Ausfall 2 Knoten verloren, d.h. es fehlt ein Knoten, um das Quorum von 4 zu erreichen (5 Available Knoten - 2 = 3 < 4). In diesem
Fall sind keine Clusteraktionen möglich und etwaige Clusterbefehle (z.B. Ressourcen starten) werden nicht ausgeführt (auch in dem Zustand, wo man 
aktuell 5 laufende Pacemakerknoten hat!) 

Anpassen kann man die Regel mit: 
`corosync-quorumtool -e` z.B. `corosync-quorumtool -e 3`

Anzeigen des Status mit -s:

.. code:: bash

    # corosync-quorumtool -s                                                                                                                                                                                              [12/24]
    Quorum information
    ------------------
    Date:             Thu Feb 16 12:05:30 2023
    Quorum provider:  corosync_votequorum
    Nodes:            5
    Node ID:          1
    Ring ID:          28808
    Quorate:          Yes

    Votequorum information
    ----------------------
    Expected votes:   5
    Highest expected: 5
    Total votes:      5
    Quorum:           3
    Flags:            Quorate WaitForAll

    Membership information
    ----------------------
        Nodeid      Votes Name
            1          1 <ip> (local)
            2          1 <ip>
            4          1 <ip>
            5          1 <ip>
            7          1 <ip>


SAPHanaSR zeigt nur ein DC an, srHook wird nicht angezeigt
==============================================================
.. index:: srHook, crm_attribute

Wenn man sicher ist, wie der Zustand des Clusters ist, kann man das auch manuell setzen:
::
    
    # Datacenter manuell setzen
    crm_attribute -n hana_<sid>_glob_sec -v DC2 -t crm_config -s SAPHanaSR   # Failover DC
    crm_attribute -n hana_<sid>_glob_prim -v DC1 -t crm_config -s SAPHanaSR  # Primary DC

    # srHook global manuell setzen
    crm_attribute -n hana_<sid>_glob_srHook -v SOK -t crm_config -s SAPHanaSR

    # srHook je Site setzen
    crm_attribute -n hana_<sid>_site_srHook_DC1 -v PRIM -t crm_config -s SAPHanaSR
    crm_attribute -n hana_<sid>_site_srHook_DC2 -v SOK -t crm_config -s SAPHanaSR
    

Support
********
.. index:: hb_report

hb_report ausführen: 
::

    hb_report -u root -f "2020/08/10 11:00" -t "2020/08/11 11:00" /tmp/hb_report_log


Konfiguration
***************

SBD als STONITH Device
=======================
SBD wird als STONITH Device verwendet. 

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
	Timeout (watchdog) : 30   <- watchdog * 2 <= msgwait 
	Timeout (allocate) : 2
	Timeout (loop)     : 5
	Timeout (msgwait)  : 90   
	==Header on disk /dev/disk/by-id/scsi-<id> is dumped



Watchdog für Storage Based Fencing
-----------------------------------

Jeder Pacemaker-Knoten prüft, ob es die angebundenen SBD Devices ansprechen kann.
Moderne Systeme haben einen Hardware-Watchdog. Dieser wird zyklisch von einem Software-Dämon zurückgesetzt. Wenn dieser 
Mechanismus unterbrochen wird, wird durch den watchdog ein SystemReset ausgeführt. Dieser Mechanismus schützt auch den 
SBD Prozess, wenn dieser "stirbt" oder aber aufgrund von i/o - Problemen nicht mehr ansprechbar ist. 

In der Lösung ist hier der ipmi_watchdog implemetiert:

.. code:: bash

    lsmod | egrep "(wd|dog|i6|iT|ibm)"
    ipmi_watchdog          32768  1
    ipmi_msghandler        49152  3 ipmi_devintf,ipmi_si,ipmi_watchdog

Das Verhalten testen kann man, indem man ein :code:`touch /dev/watchdog` oder beim softdog ein :code:`echo1> /dev/watchdog` absetzt. Das 
System sollte dann sofort fencen. 

Pacemaker Konfiguration STONITH Device
----------------------------------------
Für das STONITH Device wird eine Regel in pacemaker definiert:

.. code:: bash
    
    primitive stonith-sbd stonith:external/sbd \
            params pcmk_action_limit=-1 pcmk_delay_max=30s

pcmk_delay_max in ScaleOut 1s, in ScaleUp 30s, um zu verhindern, das sich zwei Knoten gleichzeitig "abschießen". (-> `<https://clusterlabs.org/pacemaker/doc/2.1/Pacemaker_Explained/html/fencing.html#fencing>`_)

Test Fencing
-------------
Test SBD Fencing
^^^^^^^^^^^^^^^^
SBD Starverhalten anpassen (nur für test)
  /etc/sysconfig/sbd -> SBD_STARTMODE change von always to clean (-> sbd startet nach einem Fencing nicht)
  csync2 -xv (kopieren der geänderten Konfigurationsdatei auf die anderen Knoten)
  crm cluster stop
  crm cluster start
  crm cluster status
  crm node fence node2
  sbd -d <sbd-device> list  -> zeigt an, welchen Status die einzelnen Knoten haben. Node1 sollte clear haben, node zwei "reset  node1". Da man in /etc/sysconfig/sbd definiert hat, dass
     der Startmode clean sein soll, muss man nun den Slot für Knoten 2 auf den SBD-Devices zurücksetzen durch
     sbd -d <sbd-device> message node02 clear. Dies muss für alle SBD Devices durchgeführt werden. Danach kann erst auf dem Knoten 2 der Cluster gestartet werden. 
   
Test Redfish
^^^^^^^^^^^^^^
fence_redfish --ip=<ip> --username=redfish_stonith --password=<password> --action-status --ssl-insecure
STATUS ON sollte rauskommen

Corosync
=========
Konfiguration
--------------
/etc/corosync/corosync.conf
  Update three sections:
  * totem
  * interface
  * nodelist

crm corosync edit (alternativ vi /etc/corosync/corosync.conf)

totem
^^^^^^
Bei zwei corosync Ringen soll der zweite Ring passiv sein. rrp_mode: passive

.. code:: bash

  ...
    max_messages: 20
    transport: udpu
    rrp_mode: passive   <--
    interface {
        ringnumber: 0
  ...

interface
^^^^^^^^^^
Ring0 und Ring1 eintragen

.. code:: bash

    totem:
      ...
        interface {
                ringnumber: 0
                mcastport: 5405
                ttl: 1
        }
        interface {
                ringnumber: 1
                mcastport: 5407
                ttl: 1
        }
    ...

nodelist
^^^^^^^^^

.. code::bash

    nodelist {
    ...
        node {
                ring0_addr: <IP Ring0>
                ring1_addr: <IP Ring1>
                nodeid: 1
        }
        node {
                ring0_addr: <IP Ring0>
                ring1_addr: <IP Ring1>
                nodeid: 2
        }
    ...

Mittels csync2 -xv wieder auf alle anderen Knoten kopieren

corosync-cfgtool -R    -> Reload corosync.conf auf allen Knoten
corosync-cfgtool -s    -> Show corosync configuration
crm corosync status    -> show corosync configuration


cibadmin
=========
query and edit the Pacemaker configuration, runtime information vom Cluster

 -Q --scope crm_config | grep no-quorum-policy




