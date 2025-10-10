# SAPHanaSR 

## Programme

SAPHanaSR-showAttr - Anzeige des Status HANA Cluster

Post Mortem Analyse:
SAPHanaSR-showAttr --sid <SID> --cib=cib.xml
SAPHanaSR-replay-archive --format=script hb_report_<...>.tar.bz2 | SAPHanaSR-filter --search='Hosts/<host>/roles' --filterDouble


## Konfiguration


## Pakete

```    
rpm -qa |grep -i saphanasr
SAPHanaSR-ScaleOut-doc-0.180.1-3.23.1.noarch
SAPHanaSR-ScaleOut-0.180.1-3.23.1.noarch
SAPHanaSR-angi-1.2.10-150400.9.9.2.noarch  <- next gen 
```

z.B. im ANGI Paket
```
/usr/bin/SAPHanaSR-alert-fencing
/usr/bin/SAPHanaSR-hookHelper
/usr/bin/SAPHanaSR-manageProvider
/usr/bin/SAPHanaSR-replay-archive
/usr/bin/SAPHanaSR-showAttr
/usr/lib/SAPHanaSR-angi
/usr/lib/SAPHanaSR-angi/saphana-common-lib
/usr/lib/SAPHanaSR-angi/saphana-controller-common-lib
/usr/lib/SAPHanaSR-angi/saphana-controller-lib
/usr/lib/SAPHanaSR-angi/saphana-filesystem-lib
/usr/lib/SAPHanaSR-angi/saphana-topology-lib
/usr/lib/SAPHanaSR-angi/saphana_sr_tools.py
/usr/lib/ocf
/usr/lib/ocf/resource.d
/usr/lib/ocf/resource.d/suse
/usr/lib/ocf/resource.d/suse/SAPHanaController
/usr/lib/ocf/resource.d/suse/SAPHanaFilesystem
/usr/lib/ocf/resource.d/suse/SAPHanaTopology
/usr/share/SAPHanaSR-angi
/usr/share/SAPHanaSR-angi/icons
/usr/share/SAPHanaSR-angi/icons/arrow_left_green.svg
/usr/share/SAPHanaSR-angi/icons/arrow_left_red.svg
/usr/share/SAPHanaSR-angi/icons/arrow_left_template.svg
/usr/share/SAPHanaSR-angi/icons/arrow_left_yellow.svg
/usr/share/SAPHanaSR-angi/icons/arrow_none_grey.svg
/usr/share/SAPHanaSR-angi/icons/arrow_right_green.svg
/usr/share/SAPHanaSR-angi/icons/arrow_right_red.svg
/usr/share/SAPHanaSR-angi/icons/arrow_right_template.svg
/usr/share/SAPHanaSR-angi/icons/arrow_right_yellow.svg
/usr/share/SAPHanaSR-angi/icons/back.svg
/usr/share/SAPHanaSR-angi/icons/fwd.svg
/usr/share/SAPHanaSR-angi/icons/pause.svg
/usr/share/SAPHanaSR-angi/icons/play.svg
/usr/share/SAPHanaSR-angi/icons/server_green.svg
/usr/share/SAPHanaSR-angi/icons/server_grey.svg
/usr/share/SAPHanaSR-angi/icons/server_red.svg
/usr/share/SAPHanaSR-angi/icons/server_template.svg
/usr/share/SAPHanaSR-angi/icons/server_yellow.svg
/usr/share/SAPHanaSR-angi/icons/stepBwd.svg
/usr/share/SAPHanaSR-angi/icons/stepBwd_Blue.svg
/usr/share/SAPHanaSR-angi/icons/stepBwd_Green.svg
/usr/share/SAPHanaSR-angi/icons/stepBwd_Pink.svg
/usr/share/SAPHanaSR-angi/icons/stepBwd_Purple.svg
/usr/share/SAPHanaSR-angi/icons/stepBwd_Red.svg
/usr/share/SAPHanaSR-angi/icons/stepBwd_Teal.svg
/usr/share/SAPHanaSR-angi/icons/stepFwd.svg
/usr/share/SAPHanaSR-angi/icons/stepFwd_template.svg
/usr/share/SAPHanaSR-angi/icons/suse-logo.svg
/usr/share/SAPHanaSR-angi/samples
/usr/share/SAPHanaSR-angi/samples/SAPHanaSR-upgrade-to-angi-demo
/usr/share/SAPHanaSR-angi/samples/crm_cfg
/usr/share/SAPHanaSR-angi/samples/crm_cfg/angi-ScaleUp
/usr/share/SAPHanaSR-angi/samples/crm_cfg/angi-ScaleUp/010_basics_crm.txt
/usr/share/SAPHanaSR-angi/samples/crm_cfg/angi-ScaleUp/020_resource_sbd_crm.txt
/usr/share/SAPHanaSR-angi/samples/crm_cfg/angi-ScaleUp/030_clone_top_crm.txt
/usr/share/SAPHanaSR-angi/samples/crm_cfg/angi-ScaleUp/040_clone_fil_crm.txt
/usr/share/SAPHanaSR-angi/samples/crm_cfg/angi-ScaleUp/050_clone_con_crm.txt
/usr/share/SAPHanaSR-angi/samples/crm_cfg/angi-ScaleUp/050_clone_con_fence_crm.txt
/usr/share/SAPHanaSR-angi/samples/crm_cfg/angi-ScaleUp/060_resource_ip_crm.txt
/usr/share/SAPHanaSR-angi/samples/crm_cfg/angi-ScaleUp/070_constraints_crm.txt
/usr/share/SAPHanaSR-angi/samples/global.ini_susChkSrv
/usr/share/SAPHanaSR-angi/samples/global.ini_susChkSrv_fence
/usr/share/SAPHanaSR-angi/samples/global.ini_susCostOpt
/usr/share/SAPHanaSR-angi/samples/global.ini_susHanaSR
/usr/share/SAPHanaSR-angi/samples/global.ini_susTkOver
/usr/share/SAPHanaSR-angi/susChkSrv.py 
/usr/share/SAPHanaSR-angi/susCostOpt.py
/usr/share/SAPHanaSR-angi/susHanaSR.py
/usr/share/SAPHanaSR-angi/susTkOver.py
/usr/share/doc/packages/SAPHanaSR-angi
/usr/share/doc/packages/SAPHanaSR-angi/README.md
/usr/share/licenses/SAPHanaSR-angi
/usr/share/licenses/SAPHanaSR-angi/LICENSE
/usr/share/man/man7/SAPHanaController-scale-out.7.gz
/usr/share/man/man7/SAPHanaController-scale-up.7.gz
/usr/share/man/man7/SAPHanaFilesystem.7.gz
/usr/share/man/man7/SAPHanaSR-ScaleOut.7.gz
/usr/share/man/man7/SAPHanaSR-ScaleOut_basic_cluster.7.gz
/usr/share/man/man7/SAPHanaSR-angi-scenarios.7.gz
/usr/share/man/man7/SAPHanaSR-angi.7.gz
/usr/share/man/man7/SAPHanaSR.7.gz
/usr/share/man/man7/SAPHanaSR_basic_cluster.7.gz
/usr/share/man/man7/SAPHanaSR_maintenance_examples.7.gz
/usr/share/man/man7/SAPHanaSR_upgrade_to_angi.7.gz
/usr/share/man/man7/SAPHanaTopology.7.gz
/usr/share/man/man7/ocf_suse_SAPHana.7.gz
/usr/share/man/man7/ocf_suse_SAPHanaController.7.gz
/usr/share/man/man7/ocf_suse_SAPHanaFilesystem.7.gz
/usr/share/man/man7/ocf_suse_SAPHanaTopology.7.gz
/usr/share/man/man7/susChkSrv.py.7.gz
/usr/share/man/man7/susCostOpt.py.7.gz
/usr/share/man/man7/susHanaSR.py.7.gz
/usr/share/man/man7/susHanaSrMultiTarget.py.7.gz
/usr/share/man/man7/susTkOver.py.7.gz
/usr/share/man/man8/SAPHanaSR-alert-fencing.8.gz
/usr/share/man/man8/SAPHanaSR-hookHelper.8.gz
/usr/share/man/man8/SAPHanaSR-manageProvider.8.gz
/usr/share/man/man8/SAPHanaSR-replay-archive.8.gz
/usr/share/man/man8/SAPHanaSR-show-hadr-runtimes.8.gz
/usr/share/man/man8/SAPHanaSR-showAttr.8.gz
/usr/share/man/man8/SAPHanaSR-upgrade-to-angi-demo.8.gz
``` 

### HA/DR Provider Hook Scripte

 
#### /usr/share/SAPHanaSR-angi/susHanaSR.py
The global attribute sync_state is set by regular SUSE RA monitors, based on HANA systemReplicationStatus.py. 
SAPHanaSR.py updates the cluster attribute srHook at changes of the HANA system replication status. Thus this attribute is more reliable than the attribute sync_state which is polled by RA monitors.

Scale-up with separate network links for HANA system replication and SUSE cluster can cause wrong sync_state SOK in rare cases.

Scale-up may run in monitor timeouts and show wrong sync_state SFAIL.

Scale-up needs srHook for multi-target setups, because sync_state is not multi-target aware.

The site-specific attribute srHook is set by exceptional HANA HA/DR provider events srConnectionChanged(), via SUSE hook script SAPHanaSR.py. The script is shipped since SAPHanaSR 0.154.

The srHook is defacto mandatory for scale-up.

#### /usr/share/SAPHanaSR-angi/susCostOpt.py

SAP allows to run a non-replicated instance of HANA in parallel to the replication secondary on the pre-defined failover site, e.g. a development system.
In case of failover for the production HANA the secondary HANA is promoted after the shutdown of the non-replicated HANA. The HA/DR provider method 
postTakeover() is used here. 

Starting with version 0.160.1 the SAPHanaSR package ships the hook script susCostOpt.py for changing memory limits and table preload on fail-over.

susCostOpt.py executes SQL statements to remove the memory allocation limit and switches off HANA column table preload after takeover from primary to secondary site in an SAP HANA scale-up cost-optimized scenario.

This hook script is mandatory for cost-optimised setups.

#### /usr/share/SAPHanaSR-angi/susChkSrv.py

Purpose of susChkSrv.py is to detect failing HANA indexserver processes and trigger a fast takeover to the secondary site. The cluster resource agent for HANA does not trigger a takeover to the secondary site when a failure causes an HANA process to be restarted locally by the HANA daemon.

The time needed for restarting big HANA indexservers exceeds SLA.

HA/DR provider hook srServiceStateChanged() calls the script susChkSrv.py in case the HANA indexserver process is failing.

The script can stop or kill the entire HANA instance. In consequence the cluster will initiate sr_takeover, if the system replication is fine. The script susChkSrv.py is shipped in SAPHanaSR 0.162 and SAPHanaSR-ScaleOut 0.183. 

This setup is recommended for new deployments.

Three action options: [ignore], stop, kill, fence. 

The cluster initiates a takeover only when the SR status is SOK and the PREFER_SITE_TAKEOVER parameter is  set to true. Otherwise, the cluster attempts to handle a restart locally (if feasible).

May fence node even if cluster is in maintenance. HANA scale-out does not behave well on fence. 

Logs into own trace file at master nameserver. Replaces SAP´s srServiceStateChangedHook.py.


#### /usr/share/SAPHanaSR-angi/susTkOver.py

HANA database administration tools and SUSE HA are unfortunately not seamlessly integrated so far.

However, HA/DR provider method preTakeover() informs the HA cluster about manual sr_takeover attempt. The cluster then either processes or blocks.

A first version of HADR provider hook script susTkOver.py is shipped with SAPHanaSR 0.160. It allows blocking manual sr_takeover requests (admin, 
user or 3rd party tools) while HANA is managed by the SUSE HA cluster.

Beim Takeover gibt es weitere CIB attribute:
* hana_\<sid\>_sra = [T|R|-]
* hana_\<sid\>_srah = [T|R|-]

   * T = Takeover on new primary (sr_takeover) ongoing
   * R = Registration on new secondary (sr_register) ongoing
   * \- = No action pending

/usr/share/SAPHanaSR-angi/susTkOver.py nutzt das Script **/usr/bin/SAPHanaSR-hookHelper** und prüft den Status der CIB Attribute, in welchem Zustand sich die HANA befindet:

```
USAGE:    /usr/bin/SAPHanaSR-hookHelper --sid=<SID> [--ino=<InstanceNumber>] --case <use case>
        --sid=<SID>: SID (like HA1) of the SAP system
        --ino=<InstanceNumber>: instance number (like 10) of the SAP instance
        --case=<use case>: the use case for the hook helper
                           at the moment only 'checkTakeover' and 'fenceMe' is supported
        --version: show script version
        --help:    show help
``` 





### Python Hook

In der global.ini der SAP Hana wird die Schnittstelle zum SAPHanaSR Agenten auf der pacemaker Seite konfiguriert, damit beide 
interagieren können. 

```
    mkdir -p /hana/shared/myHooks/
    cp /usr/share/SAPHanaSR-ScaleOut/SAPHanaSR.py /hana/shared/myHooks
    chown -R <sid>adm:sapsys /hana/shared/myHooks
```

Die global.ini wird um den ha_dr_provider erweitert.

 
 ```
    [ha_dr_provider_SAPHanaSR]
    provider = SAPHanaSR
    path = /hana/shared/myHooks
    execution_order = 1
    ...
```

Ob die Integration funktioniert, kann man als <sid>adm u.a. auch aus den nameserver-tracefiles erkennen: 

```
    awk  '/ha_dr_SAPHanaSR.*crm_attribute/ \
     { printf "%s %s %s %s\n",$2,$3,$5,$16 }' nameserver_<hostname>*
``` 

### /etc/sudoers

Damit der SAPHana-Agent als <sid>adm in die Pacemaker - Konfiguration reinschreiben kann (um den Status des Clusters aus HANA Sicht pacemaker mitzuteilen), benötigt <sid>adm Rechte. 
Diese werden in der /etc/sudoers definiert:

```
    # SAPHanaSR-ScaleOut needed for srHook
    Cmnd_Alias SOK = /usr/sbin/crm_attribute -n hana_<sid>_glob_srHook -v SOK -t crm_config -s SAPHanaSR
    Cmnd_Alias SFAIL = /usr/sbin/crm_attribute -n hana_<sid>_glob_srHook -v SFAIL -t crm_config -s SAPHanaSR
    <sid>adm ALL=(ALL) NOPASSWD: SOK, SFAIL
```


### /usr/lib/ocf/resource.d/suse/SAPHanaController

 (man ocf_suse_SAPHanaController)

* start / stop / monitor der SAP HANA Datenbank
* check sync Status der Systemreplikation (SOK, SWAIT, SFAIL)

Neben diesen Hauptfunktionen ist in diesem Script auch enthalten, welche Strategie präferiert wird anhand der Ausgabe von SAPHanaSR-showAttr. 
Das sind regex Ausdrücke, die im weiteren ausgewertet werden.

```
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
```

### SAPHanaSR-showAttr

Beim Neuaufsetzen des Clusters sind z.T. noch gewisse Stati nicht gesetzt. 

sudo /usr/sbin/crm_attribute -n hana_y04_glob_srHook -v SOK -t crm_config -s SAPHanaSR
sudo /usr/sbin/crm_attribute -n hana_y04_site_srHook_DC1 -v PRIM -t crm_config -s SAPHanaSR
sudo /usr/sbin/crm_attribute -n hana_y04_site_srHook_DC2 -v SOK -t crm_config -s SAPHanaSR

crm_attribute -n hana_y04_glob_sec -v DC2 -t crm_config -s SAPHanaSR


### /usr/lib/ocf/resource.d/suse/SAPHanaTopology

(man ocf_suse_SAPHanaTopology)

Das Script wertet die Rückgabe von *landscapeHostConfiguration.py* aus. 


### /usr/lib/ocf/resource.d/suse/SAPStartSrv

man SAPStartSrv
https://github.com/SUSE/SAPStartSrv-resourceAgent

## Analyse

SAPHanaSR-replay-archive  - Tool für die Analyse von hb_report
Bsp: SAPHanaSR-replay-archive --format=script hb_report_log_<hostname>.tar.bz2 2>/dev/null | SAPHanaSR-filter --filterDouble --search="clone_state|score|roles|srHook|sync_state" --showFormerValues

