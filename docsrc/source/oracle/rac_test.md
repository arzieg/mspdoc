# Oracle RAC in Azure

## Allgemein

Im Infrastrukturprodukt x86 wird RAC als kombinierte HA/DR Komponente eingesetzt zur Absicherung des Datenbankservices. In der eingesetzten Architektur wird ein Knotenausfall damit abgesichert, dieser kann durch Ausfall des Knotens selber, der Netzwerkverbindung zum Knoten oder aber durch einen kompletten Side-Verlust eines RZs resulieren. In der Cloud kann dies in der Form nicht umgesetzt werden. 

*While Oracle RAC can be used for high availability on-premises, Oracle RAC alone can't be used for high availability in the cloud. Oracle RAC only protects against instance level failures and not against rack-level or datacenter-level failures. For this reason, Oracle recommends using Oracle Data Guard with your database, whether single instance or RAC, for high availability.* [https://learn.microsoft.com/en-us/azure/virtual-machines/workloads/oracle/oracle-reference-architecture, abgerufen am 22.04.2024]

Aufgrund dieser Einschränkung, muss das RAC Template für den laborbetrieb in der Cloud modifiziert werden. 
* Zum einen gibt es keine Disks die über Storagemitteln repliziert werden, anstelle dessen wird eine ASM Replikation eingerichtet. 
* Zum anderen weicht die Cloud-Implementierung dahingehend ab, dass nur eine Instanzabsicherung getestet werden kann. 
* eth5/eth7 die im Standard als Cache Fusion Interfaces konfiguriert wurden, sind nun nicht mehr zwingend auf diesem Interface.


## Definitionen

### Subnetze

In diesem Template sind drei Subnetze zu definieren. Der Netzbereich eines Sterns umfasst 256 IPs (/24-er Netz). Dies wird unterteilt in ein /26-er Netz für das Servernetz und zwei /29-er Netze für die Oracle Cache Fusion Kommunikation. 

### VM

Erstellung der VM mittels Terraform. Ein geeigneter Typ ist anzugeben. 


## RAC Setup



### Setup der Server

TODO: beim Image soll auch das Einbinden des SMB Fileshares mit ausgeliefert werden
      https://learn.microsoft.com/en-us/azure/storage/files/storage-how-to-use-files-linux?tabs=SLES%2Csmb311

1. unter 02_virtual_machines zwei Unterordnet anlegen mit dem jeweiligen Hostnamen für RAC Knoten 1 und RAC Knoten 2
   
   Beispiel: 
   ```
   02_virtual_machines
     |_ 01_lasc53rac1
     |_ 02_lasc53rac2
    ```
     
2. Kopieren des VM Templates in die jeweiligen Verzeichnisse. Folgende Dateien sind zu editieren

2.1 backend.tf: Anpassen des Key für die tfstate
2.2 variables.tfvars: Anpassen diverser Parameter (Hostname, Image, Netze)


3. Anlegen der Disks

3a: chronyc sources zeigt auf 10.10.191.1 -> ändern

Terraform: 
  * priv3 wird angelegt, es muss priv2 heißen


4. reboot der Systeme (Prüfen im SUMA)

5. Temporärer Hack. Netzwerkinterfaces eth1 und 2 auf eth5 und 7 umbenennen. (habe ich nicht mehr gemacht!)
   Da in azure der waagent läuft und dieser hart etc/udev/rules.d/70-persistent-net.rules weglöscht, steht im github Hinweis, dass wenn ein anderer Name verwendet wird, dies nicht passiert. Also temporär dann folgendes erstellen:

   vi /etc/udev/rules.d/71-persistent-net.rules
   ```
    SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="00:22:48:5d:4e:67", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth7"
    SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="00:22:48:5d:40:7a", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth5"
   ```
    Die MAC Adressen vorher dann von eth2 und eth1 ermitteln. 
    Dies ist kein dauerhafter Zustand! https://github.com/Azure/WALinuxAgent/issues/1877: *But on Azure, the fallback to ID_NET_NAME_MAC is used by udev.
Important: I cannot recommend it, because restore of a snapshot may create interfaces with new MACs. Thus you cannot connect to an instance restored from a snapshot, since your ifcfg files does not match the new interface name, which is called with the new MAC.*



6. Hostname
  Hostname in /etc/hosts pflegen 

6a. static searchlist anpassen in /etc/resolv.conf

vi /etc/sysconfig/network/config
NETCONFIG_DNS_STATIC_SEARCHLIST="clab.azr.ez.edeka.net"
netconfig update -f

wenn dann nslookup shortname durchgeführt wird, wird der Eintrag erweitert mit dem fqdn

7. Nacharbeiten nach https://wiki.ez.edeka.net/display/PXT/Nacharbeiten+Serverinstallation


8. Multicast-Netz aufbauen und dann ein highstate durchführen. Ab diesen Zeitpunkt in den States dann ein definieres 192.168.1.x und 192.168.2.x (besser vlt. Fallback in States schreiben)

https://www.suse.com/de-de/support/kb/doc/?id=000019672 (systemd-unit)
/etc/systemd/system/edgepriv1.service
```
  [Unit]
  Description=n2n edge process for priv1
  After=network-online.target syslog.target nfw.target
  Wants=network-online.target

  [Service]
  Type=simple
  ExecStartPre=
  ExecStart=/usr/sbin/edge /etc/n2n/edgepriv1.conf
  Restart=on-abnormal
  RestartSec=5

  [Install]
  WantedBy=multi-user.target
  Alias=
```
/etc/systemd/system/edgepriv2.service
```
[Unit]
Description=n2n edge process for priv2
After=network-online.target syslog.target nfw.target
Wants=network-online.target

[Service]
Type=simple
ExecStartPre=
ExecStart=/usr/sbin/edge /etc/n2n/edgepriv2.conf
Restart=on-abnormal
RestartSec=5

[Install]
WantedBy=multi-user.target
Alias=
```

systemctl daemon-reload
systemctl enable edgepriv1.service
systemctl enable edgepriv2.service
systemctl start edgepriv1.service
systemctl start edgepriv2.service


8a. virtuelle IP anpassen an eth0 
```
lasc53rac1:/etc/sysconfig/network # cat ifcfg-eth0
CLOUD_NETCONFIG_MANAGE='yes'
IPADDR_1='10.10.53.12/26'
BOOTPROTO='dhcp+autoip'
STARTMODE='auto'
LABEL_1='virtual'
```

9. Basis SALT
   `https://wiki.ez.edeka.net/display/PXT/Nacharbeiten+Serverinstallation`

   grains anpassen: 

   ```
    template:
      profile: TEMPLATE SLES15.5 RAC Profile
      serverrole: RAC
      cluster:
        node1: lasc53rac1.clab.azr.ez.edeka.net
        node2: lasc53rac2.clab.azr.ez.edeka.net
        if-interconnect1: edge0
        if-interconnect2: edge1
      releaseversion: hasenbraeu
      sec: PROD
    ```

virtuelle IPs einmal dekonfigurieren
systemctl stop cloud-netconfig.timer
ip a d 10.10.53.12/26 dev eth0

systemctl stop cloud-netconfig.timer
ip a d 10.10.53.13/26 dev eth0

10.10.53.20 cluster-ip muss noch an ein Netzwerkinterface gebunden werden.

test : zypper se bridge-utils

-- im ersten test auch keine lösung
Cluster startet nicht. kann man diesen Workarround nutzen? 
Oracle Support Document 2391108.1 (Bug 27213224 - Deploying Exadata Software Fails At - Step 12 (Initializing Cluster Software)) can be found at: https://support.oracle.com/epmos/faces/DocumentDisplay?id=2391108.1
https://www.cyberciti.biz/tips/how-do-i-drop-or-block-attackers-ip-with-null-routes.html
lasc53rac2:~ # ip r d 169.254.169.254 via 10.10.53.1 dev eth0
lasc53rac2:~ # ip route add blackhole 169.254.169.254


5. Vorgehen nach `https://wiki.ez.edeka.net/display/PXT/xClassic+Installation+und+Migration`

5.1. In der Cloud ist die root Disk nicht als LVM erstellt -> anpassen?
5.2. wir liefern eine lvm.conf aus, die in der Cloud nicht kompatibel ist (da kein multipath) -> anpassen
5.3. lvm create überarbeiten mit den Sizes. Wenn man 32G angibt als size, fehlt ein Block. Vlt. kann man die gesamte Disk konfigurieren.
5.4. Multicast vorher anpassen, hierzu sind auch die States anzupassen, da Standard Netzwerk auf eth5/7 gelegt wird, dass muss nun geändert werden auf edge0 und 1 z.B.
Terraform 5.5. NAS bereitstellen
     manuell: https://learn.microsoft.com/en-us/azure/storage/files/storage-files-quick-create-use-linux
     public mit Reduktion auf das Netzwerk 
Terraform 5.6 SCAN Adressen bereitstellen
  Prüfen: Scan Listener Namesauflösung
      nslookup lhothrac.clab.azr.ez.edeka.net funktioniert mit den drei IP Adressen
      umgekehrt 10.10.53.20 funktioniert nicht. 
Terraform DNS Hostnamen registrieren
    lasc53rac2-vip.clab.azr.ez.edeka.net
    lasc53rac1-vip.clab.azr.ez.edeka.net
Terrafrom: VIP Adressen den NICs zuordnen
    Die VIP Adressen müssen als weitere logische IP den NICs zugeordnet werden. Ein bind per OS scheint nicht ausreichend zu sein
Beim Highstate werden die Interface interconnect auf MTU 9000 gesetzt, dass ist hierfür nzu viel. Der Eintrag muss aus /etc/sysconfig/network/ifcfg-edge0 und edge1 genommen werden. Danach rcnetwork restart


10. Netzwerk anpassen
vi /etc/sysconfig/network/ifroute-edge0
224.0.0.0/4 - - edge0
169.254.0.0/16 - - edge0
rcnetwork restart
prüfen ob route gesetzt (ip r)

für edge1 noch zu erarbeiten

10a. sysctl settings (! Achtung nach Highstate wieder neu zu setzen!)
/etc/sysctl.d/99-lunar 
net.ipv4.conf.edge0.rp_filter=2 
net.ipv4.conf.edge1.rp_filter=2 
net.ipv4.ip_forward=1  # erlaubt das routing


10b. multicast test
Schlüssel für root austauschen
/usr/bin/ssh-keygen -t rsa -b 2048 -q -N '' -f /home/oracle/.ssh/id_rsa

id_rsa.pub in die jeweilige authorized_keys
testen
ssh lasc53rac1 date && ssh lasc53rac2 date

11. Oracle RAC

6.1. Installationsdokument überarbeiten
     scp Fix noch notwendig? 
     6.6 disks identifizieren

Multicast: 
  RAC1: supernode /etc/n2n/supernode-priv1.conf
  RAC2: supernode /etc/n2n/supernode-priv2.conf
  RAC1: edge /etc/n2n/edgepriv1.conf
  RAC1: edge /etc/n2n/edgepriv2.conf
  RAC2: edge /etc/n2n/edgepriv1.conf
  RAC2: edge /etc/n2n/edgepriv2.conf

// nein, das nicht 
lasc53rac1:~ # ip r d 168.63.129.16
lasc53rac1:~ # ip r d 169.254.169.254
lasc53rac2:~ # ip r d 168.63.129.16
lasc53rac2:~ # ip r d 169.254.169.254
  / weiter oben schon gesetzt 
  route add -net 224.0.0.0/24 dev edge0
  route add -net 169.254.0.0 netmask 255.255.240.0 dev edge0
  route add -net 230.0.1.0/24 dev edge1
  route add -net 169.254.16.0 netmask 255.255.240.0 dev edge1
// ende nein 


## Troubleshooting

### GRID
Oracle Support Document 1368382.1 (Top 5 Grid Infrastructure Startup Issues) can be found at: https://support.oracle.com/epmos/faces/DocumentDisplay?id=1368382.1

HAS wollte nicht starten, relink der GRID hat geholfen
Oracle Support Document 1536057.1 (How To Relink The Oracle Grid Infrastructure Standalone (Restart) Installation Or Oracle Grid Infrastructure RAC/Cluster Installation (11.2 to 21c).) can be found at: https://support.oracle.com/epmos/faces/DocumentDisplay?id=1536057.1



### Multicast
Ein RAC benötigt ein private Netz mit Multicast. Ist dies nicht eingerichtet, funktioniert der Interconnect zw. den beiden RAC Knoten nicht.
Hinweis: Oracle Support Document 1212703.1 (Grid Infrastructure Startup During Patching, Install or Upgrade May Fail Due to Multicasting Requirement) can be found at: https://support.oracle.com/epmos/faces/DocumentDisplay?id=1212703.1

In dem Hinweis ist ein tgz - Archiv mit einem Hilfsprogramm zum Testen auf mutlicast-Fähigkeit. 
```
tar -xvf mcasttest.tgz
cd mcasttest
perl mcasttest.pl -n lasc53rac1,lasc53rac2 -i eth5,eth7
```

Interpreting the outcome of the mcasttest-tool correctly (Achtung Hinweis ist für Oracle 11):

* Should the mcasttest.pl test-tool have failed for both, the 230.0.1.0 and the 224.0.0.251 address
  you must work with your System and / or Network Administrator to enable multicast on one of the addresses.
* Should the mcasttest.pl test-tool have failed for the 230.0.1.0 address only
  Apply Patch: 9974223 or any subsequent GI PSU (fix provided GI PSU 1 and above)
* Should the mcasttest.pl test-tool have returned "success" for both, the 230.0.1.0 and the 224.0.0.251 address
  No specific patch application is required and you can proceed with the installation.  


Interfaces prüfen: 
ip maddr show                         -> zeigt die Interfaces an, die Multicast könnten
ip link show eth0 | grep MULTICAST    -> oder einfach aus dem ip link show extrahieren


Lösung: 
peer-to-peer Netz: 
https://www.pythian.com/blog/technical-track/network-multicast-support-azure
https://www.buckhill.co.uk/insider/how-to-enable-broadcast-and-multicast-on-amazon-aws-ec2/2#.VpfJ3jZs-zc
https://www.ntop.org/products/n2n/
https://github.com/ntop/n2n


Azure
https://learn.microsoft.com/en-us/answers/questions/290448/multicast-option-in-azure


### n2n manual installation

```
    zypper in git
    git clone https://github.com/ntop/n2n.git
    git checkout 3.0-stable
    zypper in kernel-devel-azure
    zypper in autoconf
    zypper in automake
    zypper in libcap-devel libpsx2
    ./autogen.sh
    ./configure
    make

    optionally install
    ------------------
    make install
```

Ansehen: 
Oracle Support Document 1212703.1 (Grid Infrastructure Startup During Patching, Install or Upgrade May Fail Due to Multicasting Requirement) can be found at: https://support.oracle.com/epmos/faces/DocumentDisplay?id=1212703.1


### n2n Konfiguration

In order to start using n2n, two elements are required:
* A supernode: it allows edge nodes to announce and discover other nodes. It must have a port publicly accessible on internet.
* edge nodes: the nodes which will be a part of the virtual networks

A virtual network shared between multiple edge nodes in n2n is called a community. A single supernode can relay multiple communities and a single computer can be part of multiple communities at the same time. An encryption key can be used by the edge nodes to encrypt the packets within their community.

Es wird im POC aufgebaut: 
Je RAC Knoten ein Supernode (S1, S2)
Je RAC Knonten zwei EdgeNodes (E11, E12, E21, E22)  E<rac><knoten>
Zwei Communities RAC1, RAC2 
Die Supernodes gehören einer Federation RAC an (also eine Art Ausfallschutz)

E11 und E21 gehören zur Community RAC1 und registrieren sich beim Supernode S1
E12 und E22 gehören zur Community RAC2 und registrieren sich beim Supernode S2

Konfigurationsfiles: 
=====================

Community File /etc/n2n/community.list
```
RACpriv1
RACpriv2
```

/etc/n2n/supernode-priv1.conf
```
-f                         <- run in foreground
-v                         <- verbose
-p 1201                    <- listen port
-l 10.10.53.30:1201    
-F RAC                     <- Federationname
-c /etc/n2n/community.list <-Communityfile
-t 5641                    <- AdminPort
```

/etc/n2n/supernode-priv2.conf
```
-f
-v
-p 1202
-l 10.10.53.30:1202    
-F RAC
-c /etc/n2n/community.list
-t 5642 
```

Edge Nodes RAC1
================

/etc/n2n/edgepriv1.conf
```
-f
-d edge0
-l 10.10.53.245:1201
-c RACpriv1
-a 192.168.1.1
-E
-r
```

/etc/n2n/edgepriv2.conf
```
-f
-d edge1
-l 10.10.53.254:1202
-c RACpriv2
-a 192.168.2.1
-E
-r
```

Edge Nodes RAC2
=================

/etc/n2n/edgepriv1.conf
```
-f
-l 10.10.53.245:1201
-c RACpriv1
-a 192.168.1.2
-E
-r
```

/etc/n2n/edgepriv2.conf
```
-f
-d edge1
-l 10.10.53.254:1202
-c RACpriv2
-a 192.168.2.2
-E
-r
```

Start Supernode
```
RAC1: supernode /etc/n2n/supernode-priv1.conf
RAC2: supernode /etc/n2n/supernode-priv2.conf
```

Start Edgenode
```
RAC1: edge /etc/n2n/edgepriv1.conf
RAC1: edge /etc/n2n/edgepriv2.conf
RAC2: edge /etc/n2n/edgepriv1.conf
RAC2: edge /etc/n2n/edgepriv2.conf
```

Mit sysctl -w net.ipv4.conf.edge0.rp_filter=2 kann man den rp_filter setzen
sysctl -w net.ipv4.ip_forward=1  # erlaubt das routing
Es gibt auch eine Option -mtu beim Edge, falls es mtu-Probleme gibt (to be done)

### RAC Routen setzen
https://dibiei.blog/2020/08/12/configuracao-de-rede-para-instalacao-de-oracle-rac-na-oci/
ip route add -net 224.0.0.0 netmask 240.0.0.0 dev edge0
ip route add -net 169.254.0.0 netmask 255.255.0.0 dev edge0

ip route add 224.0.0.0/4 dev edge0


für edge1 muss man auch noch etwas machen, das muss ausgearbeitet werden.
ip route add <network>/<netmask> via <gateway> dev <interface>


TEMPORÄR
edge -l 10.10.53.245:1201 -c RACpriv1 -a 192.168.1.1 -E -r -d edge0 -m 12:7c:68:8b:2e:b8 -f -v
edge -l 10.10.53.245:1201 -c RACpriv1 -a 192.168.1.2 -E -r -d edge0 -m d2:25:89:ec:1c:4b -f -v


## 101
echo | nc -w1 -u 127.0.0.1 5645  - Supernode Status abfragen
echo | nc -w1 -u 127.0.0.1 5644  - Edgenode Status abfragen

HA kann man mit testen mit: 
  ip link set dev eth7 down / up 


# Probleme:
AVD und xsession, Eingabe nicht möglich


# KnowHow:
DNS Namensauflösung für short-name verwendet: https://www.cyberciti.biz/faq/howto-set-dns-search-list-for-host-name-lookup/

CRS-2674: Start of 'ora.gipcd' on 'lasc53rac1' failed
  Oracle Support Document 2568395.1 (CLSRSC-119: Start of the exclusive mode cluster failed While Running root.sh While Installing Grid Infrastructure 19c) can be found at: https://support.oracle.com/epmos/faces/DocumentDisplay?id=2568395.1
  RAC-CLUSTERNAME <= 15 Buchstaben



