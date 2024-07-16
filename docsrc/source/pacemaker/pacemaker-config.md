# pacemaker config

## Stonith
https://documentation.suse.com/sle-ha/15-SP4/html/SLE-HA-all/cha-ha-fencing.html

Liste der unterstützten stonith devices: `stonith -L` oder `crm ra list stonith`
Liste der unterstützten Parameter für ein stonith device: `stonith -t external/sbd -n [-h]`  

Anzeige der configuration im pacemaker
```
crm configure show

primitive rsc_stonith_SBD stonith:external/sbd \
        params pcmk_delay_max=5 \
        meta target-role=Started
```` 

*pacemaker-fenced* daemon runs on every node in the High Availability cluster. The pacemaker-fenced instance running on the DC node receives a fencing request from the pacemaker-controld. It is up to this and other pacemaker-fenced programs to carry out the desired fencing operation.

Stonith-Plugins: 
    /usr/lib64/stonith/plugins on each node. If you installed the fence-agents package, too, the plug-ins contained there are installed in /usr/sbin/fence_*.


Test der sbd devices:
    Testmessage:
        `sbd -d  <sbd dev> message <zielhost> test`
        Im Zielsystem in /var/log/messages erscheint dann diese Information



## Fencing Delays

https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/configuring_and_managing_high_availability_clusters/assembly_configuring-fencing-configuring-and-managing-high-availability-clusters#ref_fence-delays-configuring-fencing

In einem Pacemakercluster wird prinzipiell von jenem Knoten ausgeführt, welches zu einem Quorum gehört. 
In einem 2-Knoten Cluster kann prinzipiell damit jeder Knoten den anderen killen. Mit einem Fencing Delay kann die Wahrscheinlichkeit das beide Knoten sich gegenseitig abschiessen verringer werden. Hierbei sind mehrere Strategien möglich: 

static fencing delay: 

 *Setting a static delay on one node makes that node more likely to be fenced because it increases the chances that the other node will initiate fencing first after detecting lost communication. In an active/passive cluster, setting a delay on a passive node makes it more likely that the passive node will be fenced when communication breaks down. You configure a static delay by using the pcs_delay_base cluster property. You can set this property when a separate fence device is used for each node or when a single fence device is used for all nodes.*

dynamic fencing delay: 

 *A dynamic fencing delay is random. It can vary and is determined at the time fencing is needed. You configure a random delay and specify a maximum value for the combined base delay and random delay with the pcs_delay_max cluster property. When the fencing delay for each node is random, which node is fenced is also random. You may find this feature useful if your cluster is configured with a single fence device for all nodes in an active/active design.*

priority fencing delay:

 *A priority fencing delay is based on active resource priorities. If all resources have the same priority, the node with the fewest resources running is the node that gets fenced. In most cases, you use only one delay-related parameter, but it is possible to combine them. Combining delay-related parameters adds the priority values for the resources together to create a total delay. You configure a priority fencing delay with the priority-fencing-delay cluster property. You may find this feature useful in an active/active cluster design because it can make the node running the fewest resources more likely to be fenced when communication between the nodes is lost.*

Beispiel: der Master ist an rsc_SAPHana_SR_Y03_HDB10 gebunden. Da wo der Master läuft ist die Priority bei 1000, d.h. im Fehlerfall ist die Wahrscheinlichkeit hoch jenen Knoten zu killen, wo diese Resource nicht läuft. Der zweite Knoten wartet 30 Sekunden, bevor es fencing Aktivitäten startet.
```
primitive rsc_SAPHana_SR_Y03_HDB10 ocf:suse:SAPHana \
        operations $id=rsc_SAP_Y03_HDB10-operations \
        op start interval=0 timeout=3600 \
        op stop interval=0 timeout=3600 \
        op promote interval=0 timeout=3600 \
        op monitor interval=60 role=Master timeout=700 \
        op monitor interval=61 role=Slave timeout=700 \
        params SID=Y03 InstanceNumber=10 PREFER_SITE_TAKEOVER=true DUPLICATE_PRIMARY_TIMEOUT=7200 AUTOMATED_REGISTER=TRUE \
        meta priority=1000
...
priority-fencing-delay=30
...
```

Parameter: 

pcmk_delay_base: Anzahl Sekunden bevor ein fencing initiiert wird, statisches Fencing

pcmk_delay_max: Anzahl Sekunden bevor ein fencing initiiert wird, dynamisches Fencing

|                        | pcmk_delay_base is set  |  pcmk_delay_max is set    |
|------------------------|-------------------------|---------------------------|
|pcmk_delay_base is set  |                         |  Gesamtdelay = max ( Static Delay + random (delay), pcmk_delay_max) |
|pcmk_delay_max is set   | Gesamtdelay=delay_base  |  |

To specify different values for different nodes, you map the host names to the delay value for that node using a similar syntax to pcmk_host_map. For example, node1:0;node2:10s would use no delay when fencing node1 and a 10-second delay when fencing node2.
Wenn priority-fencing-delay gesetzt ist und einer oder beide anderen Parameter auch, wird das Gesamtdelay um priority-fencing-delay erhöht. 
Only fencing scheduled by Pacemaker itself observes fencing delays
Some individual fence agents implement a delay parameter, with a name determined by the agent and which is independent of delays configured with a pcmk_delay_* property. If both of these delays are configured, they are added together and would generally not be used in conjunction.

Test:




