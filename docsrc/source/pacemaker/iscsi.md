# iscsi 

## Löschen der iscsi Devices

Kill der ISCSI session: ``iscsiadm -m node -T <iqn> -p <ip address>:<port number> -u``

Node löschen: ``iscsiadm -m node -o delete -T <iqn>``

Löschen des Targets von der ISCSI discovery database: ``iscsiadm -m discoverydb -t sendtargets -p <IP>:<port> -o delete``

Überprüfen, ob noch Sesssions aktiv sind:  ``iscsiadm -m session``

Prüfen und ggfs. löschen der Nodes unter:  ``/etc/iscsi/nodes`` und ``/etc/iscsi/send_targets``

Rescan der iscsi Nodes mit aktiver Session:  ``iscsiadm -m node -R``

Rescan der iscsi Nodes ``iscsiadm -m discovery -t sendtargets -p <portal>:<port|3260>``

To safely add new targets/portals or delete old ones, use the -o new or -o delete options, respectively. For example, 
to add new targets/portals without overwriting /var/lib/iscsi/nodes AND to delete /var/lib/iscsi/nodes entries that the target 
did not display during discovery, use: ``iscsiadm -m discovery -t st -p target_IP -o delete -o new``


## Login / logout

Login/logout:
  ``iscsiadm -m node -p <portal>:<port|3260> -T <target>:lunXX --login | --logout ``

## Restore
`restoreconfig savefile=/etc/target/saveconfig.json.sles15.laszvc01 clear_existing=Falserestoreconfig savefile=/etc/target/saveconfig.json.sles15.laszvc01 clear_existing=False`
