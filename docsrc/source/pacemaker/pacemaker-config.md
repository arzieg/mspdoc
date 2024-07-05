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
