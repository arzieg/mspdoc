# Nodemanagement

## Add Worker Node
https://learning.sap.com/learning-journeys/setting-up-high-availability-and-disaster-recovery-for-sap-hana/adding-a-host-to-a-scale-out-system_a0558ee3-68ca-45a1-964a-4054b24a592c

Von einem laufenden HANA Node

hdb10y04-1002:/hana/shared/Y04/hdblcm # ./hdblcm


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

Enter selected action index [19]: 2

                                                                                                                                                                                                                 System Properties:
Y04 /hana/shared/Y04 HDB_ALONE
        HDB10
        version: 2.00.079.01.1728370356
        hosts: lavdb10y04101 (Database Worker (worker)), lavdb10y04102 (Database Worker (worker))
        edition: SAP HANA Database
        plugins: afl,rtl

Enter comma-separated host names to add: lavdb10y04103
Enter Root User Name For Remote Hosts [root]:
Collecting information from host 'lavdb10y04103'...
Information collected from host 'lavdb10y04103'.

Select roles for host 'lavdb10y04103':
Index | Host Role                | Description
  -------------------------------------------------------------------
  1     | worker                   | Database Worker   <--
  2     | standby                  | Database Standby  
  3     | extended_storage_worker  | Dynamic Tiering Worker
  4     | extended_storage_standby | Dynamic Tiering Standby
  5     | ets_worker               | Accelerator for SAP ASE Worker
  6     | ets_standby              | Accelerator for SAP ASE Standby
  7     | streaming                | Streaming Analytics
  8     | xs_worker                | XS Advanced Runtime Worker
  9     | xs_standby               | XS Advanced Runtime Standby

Enter comma-separated list of selected indices [1]:                                        
Enter comma-separated list of selected indices [1]: 2
Enter Host Failover Group for host 'lavdb10y04103' [default]:
Enter Worker Group for host 'lavdb10y04103' [default]:
Enter System Administrator (y04adm) Password:


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
      Certificate Host Names: lavdb10y04103 -> hdb10y04-1003.lunarlab.edeka.net
      Do not Modify '/etc/sudoers' File: No
      Ignore: <not defined>
   Additional Hosts
      lavdb10y04103
         Role: Database Standby (standby)
         High-Availability Group: default
         Worker Group: default
         Storage Partition: N/A
   Log File Locations
      Log directory: /var/tmp/hdb_Y04_hdblcm_add_hosts_2025-02-07_14.40.53
      Trace location: /var/tmp/hdblcm_2025-02-07_14.40.53_2433282.trc

Do you want to continue? (y/n):                                                

Adding Remote Hosts to the SAP HANA Database System
  Adding additional host...
  Adding host 'lavdb10y04103'...
    lavdb10y04103:  Adding host 'lavdb10y04103' to instance '10'...
    lavdb10y04103:  Starting SAP HANA Database...
    lavdb10y04103:    Starting 4 processes on host 'lavdb10y04103' (standby):
    lavdb10y04103:      Starting on 'lavdb10y04103' (standby): hdbdaemon, hdbcompileserver, hdbnameserver, hdbpreprocessor
    lavdb10y04103:      Starting on 'lavdb10y04103' (standby): hdbdaemon, hdbwebdispatcher
    lavdb10y04103:    All server processes started on host 'lavdb10y04103' (standby).
Updating SAP HANA Database Instance Integration on Remote Hosts...
  Updating SAP HANA Database instance integration on host 'lavdb10y04103'...
Updating Component List...
Additional hosts added to the SAP HANA Database System
Log file written to '/var/tmp/hdb_Y04_hdblcm_add_hosts_2025-02-07_15.00.28/hdblcm.log' on host 'hdb10y04-1002'.






SELECT * FROM M_TABLE_PERSISTENCE_LOCATIONS;
SELECT * FROM M_CS_PARTITIONS;

SELECT HOST, HOST_NAME FROM M_HOST_INFORMATION;

SELECT T.SCHEMA_NAME, T.TABLE_NAME, L.HOST, L.PORT
FROM M_TABLES T
JOIN M_TABLE_PERSISTENCE_LOCATIONS L ON T.TABLE_OID = L.TABLE_OID
WHERE L.HOST = '<specific_hostname>';

### hdbuserstore ist anzupassen mit neuen Nodes

## Remove Node
https://learning.sap.com/learning-journeys/setting-up-high-availability-and-disaster-recovery-for-sap-hana/removing-a-host-from-a-scale-out-system_f5a78887-dd28-44f3-98b5-b95d968a8a75



Anmelden am TENANT!

```
call SYS.UPDATE_LANDSCAPE_CONFIGURATION( 'SET REMOVE','<host>' );
call REORG_GENERATE(2,'');
select * from SYS.REORG_STEPS;
call REORG_EXECUTE(?);

hdb10y04-1003:y04adm> python ./landscapeHostConfiguration.py
| Host          | Host     | Host    | Failover | Remove             | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host   | Host   | Worker  | Worker  | |               | Active   | Status  | Status   | Status             | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config | Actual | Config  | Actual  | |               |          |         |          |                    | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles  | Roles  | Groups  | Groups  | | ------------- | -------- | ------- | -------- | ------------------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------ | ------ | ------- | ------- | | lavdb10y04101 | yes      | ok      |          |                    |         1 |         1 | default  | default  | master 1   | master     | worker      | master      | worker | worker | default | default | | lavdb10y04102 | yes      | ok      |          |                    |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker | worker | default | default | | lavdb10y04103 | starting | warning |          | reorg not required |         3 |         3 | default  | default  | master 3   | slave      | worker      | slave       | worker | worker | default | default |                                                                                                                                                      ``` 



hdbuserstore ist anzupassen mit neuen Nodes



### Remove StandBy

Als User root:

``` 
cd /hana/shared/Y04/hdblcm
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
mv /usr/sap/ABC/home/.hdb/<running host> /usr/sap/Y04/home/.hdb/<host>

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

