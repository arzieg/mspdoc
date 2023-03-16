.. _iscsi:

##########
iscsi 
##########

Löschen der iscsi Devices
==========================

Kill der ISCSI session:
  ``iscsiadm -m node -T <iqn> -p <ip address>:<port number> -u``
Node löschen:
  ``iscsiadm -m node -o delete -T <iqn>``
Löschen des Targets von der ISCSI discovery database
  ``iscsiadm -m discoverydb -t sendtargets -p <IP>:<port> -o delete``
Überprüfen, ob noch Sesssions aktiv sind
  ``iscsiadm -m session``
Pürfen und ggfs. löschen der Nodes unter
  ``/etc/iscsi/nodes`` und ``/etc/iscsi/send_targets``
