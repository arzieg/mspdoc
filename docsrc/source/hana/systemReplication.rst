.. _hana_sr:

##########################
HANA SystemReplication
##########################

3203165 - Startup fails at error 'nameserver: Couldn't find own nameserver in topology'
========================================================================================

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

