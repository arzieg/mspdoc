# Nodemanagement

## Add Worker Node
https://learning.sap.com/learning-journeys/setting-up-high-availability-and-disaster-recovery-for-sap-hana/adding-a-host-to-a-scale-out-system_a0558ee3-68ca-45a1-964a-4054b24a592c


1. Backup muss funktionieren, ansonsten im Lab Probleme. 

2. Von einem laufenden HANA Node

hdb10abc-1002:/hana/shared/ABC/hdblcm # ./hdblcm


SAP HANA Lifecycle Management - SAP HANA Database 2.00.079.01.1728370356
************************************************************************

  1     | add_host_roles             | Add Host Roles
  2     | add_hosts                  | Add Hosts to the SAP HANA Database System  <---
  3     | check_installation         | Check SAP HANA Database Installation
  4     | configure_internal_network | Configure Inter-Service Communication
  5     | configure_sld              | Configure System Landscape Directory Registration
  6     | extract_components         | Extract Components
  7     | print_component_list       | Print Component List
  8     | print_detected_components  | Print Detected Components
  9     | remove_host_roles          | Remove Host Roles
  10    | remove_hosts               | Remove Hosts from the SAP HANA Database System
  11    | rename_system              | Rename the SAP HANA Database System
  12    | uninstall                  | Uninstall SAP HANA Database Components
  13    | unregister_instance        | Unregister the SAP HANA Database Instance
  14    | unregister_system          | Unregister the SAP HANA Database System
  15    | update                     | Update the SAP HANA Database System
  16    | update_component_list      | Update Component List
  17    | update_components          | Install or Update Additional Components
  18    | update_host                | Update the SAP HANA Database Instance Host integration
  19    | exit                       | Exit (do nothing)


System Properties:
ABC /hana/shared/ABC HDB_ALONE
        HDB10
        version: 2.00.079.01.1728370356
        hosts: labc10host101 (Database Worker (worker)), labc10host102 (Database Worker (worker))
        edition: SAP HANA Database
        plugins: afl,rtl

Enter comma-separated host names to add: labc10host103
Enter Root User Name For Remote Hosts [root]:
Collecting information from host 'labc10host103'...
Information collected from host 'labc10host103'.

Select roles for host 'labc10host103':

  Index | Host Role                | Description
  -------------------------------------------------------------------
  1     | worker                   | Database Worker  <---
  2     | standby                  | Database Standby
  3     | extended_storage_worker  | Dynamic Tiering Worker
  4     | extended_storage_standby | Dynamic Tiering Standby
  5     | ets_worker               | Accelerator for SAP ASE Worker
  6     | ets_standby              | Accelerator for SAP ASE Standby
  7     | streaming                | Streaming Analytics
  8     | xs_worker                | XS Advanced Runtime Worker
  9     | xs_standby               | XS Advanced Runtime Standby

Enter comma-separated list of selected indices [1]: 1
Enter Host Failover Group for host 'labc10host103' [default]:
Enter Storage Partition Number for host 'labc10host103' [<<assign automatically>>]:
Enter Worker Group for host 'labc10host103' [default]:
Enter System Administrator (abcadm) Password:

Summary before execution:
=========================

Add Hosts to SAP HANA Database System
   Add Hosts Parameters
      Skip all SAP Host Agent calls: No
      Remote Execution: ssh
      Enable the installation or upgrade of the SAP Host Agent: Yes
      Auto Initialize Services: Yes
      Install SSH Key: Yes
      Root User Name For Remote Hosts: root
      Do not start added hosts and do not start SAP HANA Database System: No
      Certificate Host Names: labc10host103 -> hdb10abc-1003.lunarlab.edeka.net
      Do not Modify '/etc/sudoers' File: No
      Ignore: <not defined>
   Additional Hosts
      labc10host103
         Role: Database Worker (worker)
         High-Availability Group: default
         Worker Group: default
         Storage Partition: <<assign automatically>>
   Log File Locations
      Log directory: /var/tmp/hdb_ABC_hdblcm_add_hosts_2025-02-07_20.17.57
      Trace location: /var/tmp/hdblcm_2025-02-07_20.17.57_3617599.trc

Do you want to continue? (y/n): y

Adding Remote Hosts to the SAP HANA Database System
  Adding additional host...
  Adding host 'labc10host103'...
    labc10host103:  Adding host 'labc10host103' to instance '10'...
    labc10host103:  Starting SAP HANA Database...
    labc10host103:    Starting 1 process on host 'labc10host103' (worker):
    labc10host103:      Starting on 'labc10host103' (worker): hdbdaemon
    labc10host103:    Starting 4 processes on host 'labc10host103' (worker):
    labc10host103:      Starting on 'labc10host103' (worker): hdbdaemon, hdbcompileserver, hdbnameserver, hdbpreprocessor
    labc10host103:      Starting on 'labc10host103' (worker): hdbdaemon, hdbwebdispatcher, hdbindexserver (ABC)
    labc10host103:      Starting on 'labc10host103' (worker): hdbdaemon, hdbwebdispatcher
    labc10host103:    All server processes started on host 'labc10host103' (worker).
Updating SAP HANA Database Instance Integration on Remote Hosts...
  Updating SAP HANA Database instance integration on host 'labc10host103'...
Updating Component List...
Additional hosts added to the SAP HANA Database System
Log file written to '/var/tmp/hdb_ABC_hdblcm_add_hosts_2025-02-07_20.17.57/hdblcm.log' on host 'hdb10abc-1001'.


1. hdbuserstore ist anzupassen mit neuen Nodes

## Remove Node
https://learning.sap.com/learning-journeys/setting-up-high-availability-and-disaster-recovery-for-sap-hana/removing-a-host-from-a-scale-out-system_f5a78887-dd28-44f3-98b5-b95d968a8a75

1. Anmelden am TENANT!

```
SELECT * FROM SYS.REORG_STEPS;
SELECT * FROM SYS.M_LANDSCAPE_HOST_CONFIGURATION;

call SYS.UPDATE_LANDSCAPE_CONFIGURATION( 'SET REMOVE','<host>' );
call REORG_GENERATE(2,'');   <-- ab hier schon landscapeHostConfiguration aufrufen, das reicht eigentlich schon, um das System wieder rauszunehmen, sofern keine Tabellen verteilt wurden. 
select * from SYS.REORG_STEPS;

call REORG_EXECUTE(?);

-> reorg_id eine Nummer

select * from reorg_overview where reorg_id = '195';  <- wann ist der reorg gestartet
select count(*) from reorg_steps where reorg_id='195';  <- Anzahl der reorg steps (HANA geht alle Tabellen durch und prüft, ob etwas zu tun ist, auch wenn worker nur kurz reingenommen wurde, das kann sehr lange dauern. In M. wurde ein Tabellenreorg durchgeführt)

REORG_ID,STATUS,START_DATE,END_DATE,USER,ALGORITHM_ID,PARAMETERS
195,"FINISHED","2025-04-04 12:18:11.278000000","2025-04-04 15:27:39.698000000","SYSTEM",2,""


select count(*) from reorg_steps where reorg_id='195' and status='FINISHED'; <- Anzahl der beendeten Reorg-Steps



# Anzeige wie viele Jobs noch laufen müssen
select IFNULL("STATUS", 'PENDING'), count(*) from REORG_STEPS where reorg_id=(SELECT MAX(REORG_ID) from REORG_OVERVIEW) group by "STATUS";



SELECT * FROM SYS.REORG_GENERATE_OVERVIEW;


hdb10abc-1003:abcadm> python ./landscapeHostConfiguration.py
| Host          | Host     | Host    | Failover | Remove             | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host   | Host   | Worker  | Worker  | |               | Active   | Status  | Status   | Status             | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config | Actual | Config  | Actual  | |               |          |         |          |                    | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles  | Roles  | Groups  | Groups  | | ------------- | -------- | ------- | -------- | ------------------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------ | ------ | ------- | ------- | | labc10host101 | yes      | ok      |          |                    |         1 |         1 | default  | default  | master 1   | master     | worker      | master      | worker | worker | default | default | | labc10host102 | yes      | ok      |          |                    |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker | worker | default | default | | labc10host103 | starting | warning |          | reorg not required |         3 |         3 | default  | default  | master 3   | slave      | worker      | slave       | worker | worker | default | default | 
``` 

2. hdlcm remove Node
3. hdbuserstore ist anzupassen mit neuen Nodes


### Remove StandBy

Als User root:

``` 
cd /hana/shared/ABC/hdblcm
hdblcm

Choose an action

  Index | Action                     | Description
  -------------------------------------------------------------------------------------------
  1     | add_host_roles             | Add Host Roles
  2     | add_hosts                  | Add Hosts to the SAP HANA Database System
  3     | check_installation         | Check SAP HANA Database Installation
  4     | configure_internal_network | Configure Inter-Service Communication
  5     | configure_sld              | Configure System Landscape Directory Registration
  6     | extract_components         | Extract Components
  7     | print_component_list       | Print Component List
  8     | print_detected_components  | Print Detected Components
  9     | remove_host_roles          | Remove Host Roles
  10    | remove_hosts               | Remove Hosts from the SAP HANA Database System   <--
  11    | uninstall                  | Uninstall SAP HANA Database Components
  12    | update                     | Update the SAP HANA Database System
  13    | update_component_list      | Update Component List
  14    | update_components          | Install or Update Additional Components
  15    | update_host                | Update the SAP HANA Database Instance Host integration
  16    | exit                       | Exit (do nothing)


Choose Server 

Keep System Administrator User [n]: 
...
```



## readd Node (nur wenn dieser noch registriert ist im Landscape SAP HANA)

### Hostagent Install

```
/root/tmp_hostagent_install/global/hdb/saphostagent_setup/saphostexec -install
```

### Anlegen /usr/sap/sapservices

```
#!/bin/sh
limit.descriptors=1048576
systemctl --no-ask-password start SAP<SID>_<SYSNR> # sapstartsrv pf=/usr/sap/ABC/SYS/profile/ABC_HDB10_<logicalhostname>
```

Permissions:

```
chmod 755 /usr/sap/sapservices
chown root:sapsys /usr/sap/sapservices
```

### Sync usr-home

Kopie von einem laufenden Knoten

```
rsync -az -e ssh --progress --perms --exclude .snapshot <running logicalhost>:/usr/sap/ABC/ /usr/sap/ABC/
mv /usr/sap/ABC/home/.hdb/<running host> /usr/sap/ABC/home/.hdb/<host>

mkdir /var/lib/hdb
chmod 775 /var/lib/hdb
chown root:sapsys /var/lib/hdb
```

### Hook DIR für pacemaker Agenten (noch notwendig?)

Kopieren wir auch vom laufenden Knoten

```
scp -rp <running host>:/usr/share/SAPHanaSR-ScaleOut /usr/share/
chown -R abcadm:sapsys /usr/share/SAPHanaSR-ScaleOut
chmod 755 /usr/share/SAPHanaSR-ScaleOut
```

/hana/shared/suse_cluster anlegen

```
mkdir /hana/shared/suse_cluster
chown root:root /hana/shared/suse_cluster
chmod 750 /hana/shared/suse_cluster
```

### Instance Profil anlegen

```
cd /usr/sap/ABC/SYS/profile
cp ABC_HDB10_<loghostname_node2> ABC_HDB10_<logicalhostname>
vi ABC_HDB10_<logicalhostname> und logischen Hostnamen anpassen
```

```
mkdir /usr/sap/ABC/HDB10/<logicalhostname>
```

## Test

```
mount -t nfs lasc58dc1.clab.azr.ez.edeka.net:/hana/shared /hana/shared
mount -t nfs lasc58dc1.clab.azr.ez.edeka.net:/hana/data /hana/data
mount -t nfs lasc58dc1.clab.azr.ez.edeka.net:/hana/log /hana/log
/etc/hosts pflegen

groupadd -g 1000 sapsys
groupadd -g 6030 y10shm
useradd -m -d /usr/sap/Y10/home -s /bin/sh -c "SAP HANA Database System Administrator" -u 5000 -g 1000 -G y10shm y10adm

mkdir /hana/data/Y10
mkdir /hana/log/Y10
chown -R y10adm:sapsys /hana/data
chown -R y10adm:sapsys /hana/log



HOSTAGENTFILE="/hana/shared/Y10/software/repo/03_repositories/HANA_Database/HANA2_rev59_14/extracted/SAP_HANA_DATABASE/server/HOSTAGENT.TGZ"
HOSTAGENTDIR="/root/tmp_hostagent_install"
HOSTAGENTSETUPDIR="$HOSTAGENTDIR/global/hdb/saphostagent_setup"
mkdir ${HOSTAGENTDIR} 
tar -zxvf ${HOSTAGENTFILE} -C${HOSTAGENTDIR} 
cd $HOSTAGENTSETUPDIR
$HOSTAGENTSETUPDIR/saphostexec -install
cd
rm -rf ${HOSTAGENTDIR}

/etc/ssh/sshd_config
PasswordAuthentication yes
systemctl restart sshd

Auf dem Zielsystem temporär 
cd ~/.ssh # mv authorized_keys authorized_keys.bak


```
