# HANA SystemReplication

## Disable HANA SystemReplication

On the secondary System:

```
sapcontrol -nr <instance_number> -function StopSystem HDB
hdbnsutil -sr_unregister
sapcontrol -nr <instance_number> -function StartSystem HDB
```

*In cases where system replication is out of sync and you just need to re-register the initial secondary system, unregister is not required - simply use the command hdbnsutil -sr_register. For details of the two cases where hdbnsutil -sr_unregister is required see SAP Note 1945676 - Correct usage of hdbnsutil -sr_unregister.*


On the primary System:

```
hdbnsutil -sr_disable
```


## HANA Network Configuration for SAP HANA System Replication

https://www.sap.com/documents/2016/06/18079a1c-767c-0010-82c7-eda71af511fa.html

If nothing else is configured during the installation, the hostnames known to the os are used as HANA host - called internal hostnames.

For all communication between the SAP HANA Services these internal host names are used (for internal as well as system replication)

### virtual hostnames

during the installation you could specify virtual hostnames: `hdblcm ... --hostname=<virtualhostname>`

The \<virtualhostname\> will be used as the HANA internal hostname

**Find out the internal hostnames**

/usr/sap/SID/HDBnr/**hostname**/sapprofile.ini
/usr/sap/SID/SYS/profile/SID_HDBnr_**hostname**

### internal hostname resolution

ScaleOut cann run with internal network communication. `hdblcm ... --internal_network=192.168.1.0/20`

in the global.ini

```
[communication]
listeninterface=.internal

[internal_hostname_resolution]
192.168.1.1=host1
192.168.1.2=host2
...
```

If set in global.ini HANA will use this settings befor it use the /etc/hosts

### SR Network Configuration

Changes regarding network used by SAP HANA SR to global.ini must be done prior to registering the secondary, because -sr_register uses this mapping

#### SR Hostname Resolution

by default, using internal host names

you can define a separate network

Wenn auf beiden Seiten der selbe Hostname verwendet wird für die interne Kommunikation, dann muss bei eine SR mit virtuellen Hostnamen gearbeitet werden, die sich unterscheiden müssen.

global.ini:
```
[system_replication_hostname_resoultion]
192.168.0.1=host1-dc1
192.168.1.1=host1-dc2
...
```

The entries in the system_replication_hostname_resolution section is used in combination with the listeninterface parameter in the system_replication_communication section. The following combination are possible

|system_replication_communication listeninterface|system_replication_hostname_resolution|info|
|------------------------------------------------|--------------------------------------|----|
|.global                                         |no mapping specified                  |the default network is used for sr aka public network. If you use public network instead of separate network, you must secure this connection with add. mesasures such as firewall or vpn or ssl|
|.global                                         |entries for prim. and sec. hosts      |a separate network is used for sr communication|
|.internal                                       |entries for prim. and sec. hosts      |a separate network is used for sr communication. Incoming requests on the public interfaces are rejected|

global.ini DC1:
```
[communication]                                                                                                                                                   
listeninterface = .internal

[internal_hostname_resoltion]
192.168.1.1=site1host1
192.168.1.2=site1host2
192.168.1.3=site1host3

[system_replication_hostname_resolution]
10.5.1.1=site1host1
10.5.1.2=site1host2
10.5.1.3=site1host3
10.5.2.1=site2host1
10.5.2.2=site2host2
10.5.2.3=site2host3
```

global.ini DC2:
```
[communication]                                                                                                                                                   
listeninterface = .internal

[internal_hostname_resoltion]
192.168.1.1=site2host1
192.168.1.2=site2host2
192.168.1.3=site2host3

[system_replication_hostname_resolution]
10.5.1.1=site1host1
10.5.1.2=site1host2
10.5.1.3=site1host3
10.5.2.1=site2host1
10.5.2.2=site2host2
10.5.2.3=site2host3
```







## 3203165 - Startup fails at error 'nameserver: Couldn't find own nameserver in topology'


Fehlermeldung: Couldn't find own nameserver in topology (lavdb10y04101:31001), insb. nach einem harten Reset der SystemReplication

If you want to keep HANA system replication and start it as secondary site, you can solve this error by registering it again via "hdbnsutil -sr_register...".
Otherwise, you can follow below steps to start up this HANA instance.

1. Make sure the system is fully shutdown. If any service is still running, you may corrupt the topology and there is no way to recover it.

2. Then run the following command as <sid>adm user on former primary system:

hdbnsutil -exportTopology myExportedTopology.txt

3. Open the generated myExportedTopology.txt and replace any entry of hostname with the current hostname.

4. Afterwards import the fixed topology file using command on target system:

hdbnsutil -importTopology myExportedTopology.txt

5. Now start the HANA instance

