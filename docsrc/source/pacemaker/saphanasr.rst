.. _saphanasr:

##########
SAPHanaSR 
##########


Programme
**********
SAPHanaSR-showAttr - Anzeige des Status HANA Cluster

Post Mortem Analyse:
SAPHanaSR-showAttr --sid <SID> --cib=cib.xml
SAPHanaSR-replay-archive --format=script hb_report_<...>.tar.bz2 | SAPHanaSR-filter --search='Hosts/<host>/roles' --filterDouble


Konfiguration
*****************

Pakete
=======

 .. code:: bash
    
    rpm -qa |grep -i saphanasr
    SAPHanaSR-ScaleOut-doc-0.180.1-3.23.1.noarch
    SAPHanaSR-ScaleOut-0.180.1-3.23.1.noarch

Python Hook
=============
In der global.ini der SAP Hana wird die Schnittstelle zum SAPHanaSR Agenten auf der pacemaker Seite konfiguriert, damit beide 
interagieren können. 

 .. code:: bash

    mkdir -p /hana/shared/myHooks/
    cp /usr/share/SAPHanaSR-ScaleOut/SAPHanaSR.py /hana/shared/myHooks
    chown -R <sid>adm:sapsys /hana/shared/myHooks

Die global.ini wird um den ha_dr_provider erweitert.

 .. code:: bash

    [ha_dr_provider_SAPHanaSR]
    provider = SAPHanaSR
    path = /hana/shared/myHooks
    execution_order = 1
    ...

Ob die Integration funktioniert, kann man als <sid>adm u.a. auch aus den nameserver-tracefiles erkennen: 

 .. code:: bash

    awk  '/ha_dr_SAPHanaSR.*crm_attribute/ \
     { printf "%s %s %s %s\n",$2,$3,$5,$16 }' nameserver_<hostname>*
    

/etc/sudoers
==============
Damit der SAPHana-Agent als <sid>adm in die Pacemaker - Konfiguration reinschreiben kann (um den Status des Clusters aus HANA Sicht pacemaker mitzuteilen), benötigt <sid>adm Rechte. 
Diese werden in der /etc/sudoers definiert:

 .. code:: bash

    # SAPHanaSR-ScaleOut needed for srHook
    Cmnd_Alias SOK = /usr/sbin/crm_attribute -n hana_<sid>_glob_srHook -v SOK -t crm_config -s SAPHanaSR
    Cmnd_Alias SFAIL = /usr/sbin/crm_attribute -n hana_<sid>_glob_srHook -v SFAIL -t crm_config -s SAPHanaSR
    <sid>adm ALL=(ALL) NOPASSWD: SOK, SFAIL



/usr/lib/ocf/resource.d/suse/SAPHanaController
===============================================
 (man ocf_suse_SAPHanaController)

* start / stop / monitor der SAP HANA Datenbank
* check sync Status der Systemreplikation (SOK, SWAIT, SFAIL)

Neben diesen Hauptfunktionen ist in diesem Script auch enthalten, welche Strategie präferiert wird anhand der Ausgabe von SAPHanaSR-showAttr. 
Das sind regex Ausdrücke, die im weiteren ausgewertet werden.

 .. code:: bash

      SCORING_TABLE_PREFERRED_SITE_TAKEOVER=(
       "[234]:P:master[123]:master     .*          150"
       "[234]:P:master[123]            .*          140"
       "[234]:P:master[123]:slave:.*:standby      .*          115"
       "[234]:P:master[123]:slave      .*          110"
       "[015]:P:master[123]:           .*           70"
       "[0-9]:P:master[123]:*:standby  .*           60"
       "[0-9]:P:slave:                 .*       -10000"
       "[234]:S:master[123]:master     SOK         100"
       "[234]:S:master[123]:slave      SOK          80"
       "[015]:S:master[123]:           SOK          70"
       "[0-9]:S:master[124]:*:standby  SFAIL    -22100"
       "[0-9]:S:slave:                 SOK      -12200"
       "[0-9]:S:slave:                 SFAIL    -22200"
       "[0-9]:S:                       .*       -32300"
       ".*                             .*       -33333"

      SCORING_TABLE_PREFERRED_LOCAL_RESTART=(
        ...
        
      SCORING_TABLE_PREFERRED_NEVER=(
      ...

      SCORING_TABLE_PREFERRED_AGGRESSIVE=(
      ...


SAPHanaSR-showAttr
==================
Beim Neuaufsetzen des Clusters sind z.T. noch gewisse Stati nicht gesetzt. 

sudo /usr/sbin/crm_attribute -n hana_y04_glob_srHook -v SOK -t crm_config -s SAPHanaSR
sudo /usr/sbin/crm_attribute -n hana_y04_site_srHook_DC1 -v PRIM -t crm_config -s SAPHanaSR
sudo /usr/sbin/crm_attribute -n hana_y04_site_srHook_DC2 -v SOK -t crm_config -s SAPHanaSR



/usr/lib/ocf/resource.d/suse/SAPHanaTopology
=============================================
(man ocf_suse_SAPHanaTopology)

Das Script wertet die Rückgabe von *landscapeHostConfiguration.py* aus. 


/usr/lib/ocf/resource.d/suse/SAPStartSrv
=========================================
man SAPStartSrv
https://github.com/SUSE/SAPStartSrv-resourceAgent

Analyse
========
SAPHanaSR-replay-archive  - Tool für die Analyse von hb_report
Bsp: SAPHanaSR-replay-archive --format=script hb_report_log_<hostname>.tar.bz2 2>/dev/null | SAPHanaSR-filter --filterDouble --search="clone_state|score|roles|srHook|sync_state" --showFormerValues

