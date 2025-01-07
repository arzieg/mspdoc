# Podman Pacemaker

Bsp. mit SUSE Container: 

/etc/containers/registry.conf erweitern um registry.suse.com.

`unqualified-search-registries = [... "registry.suse.com"]`


p pull registry.fedoraproject.org/fedora-iot
oder
p pull quay.io/fedora/fedora-bootc:latest

container laufen auf dem selben Host

```
podman network create -d bridge --subnet 10.0.65.0/24 pub_nw
podman network create -d bridge --subnet=10.0.62.0/24 --gateway=10.0.62.1 pcm_nw
podman network create -d bridge --subnet=10.0.61.0/24 ring1_nw
podman network create -d bridge --subnet=10.0.63.0/24 ring2_nw
```

Fehler mit dem Netzwerk CLI . 
```
WARN[0000] Error validating CNI config file /home/arne/.config/cni/net.d/pub1_nw.conflist: [plugin firewall does not support config version "1.0.0"] 
WARN[0000] Error validating CNI config file /home/arne/.config/cni/net.d/ring1_nw.conflist: [plugin firewall does not support config version "1.0.0"] 
WARN[0000] Error validating CNI config file /home/arne/.config/cni/net.d/ring2_nw.conflist: [plugin firewall does not support config version "1.0.0"] 
```

Lösung: https://www.michaelmcculley.com/updating-cni-plugins-for-podman-a-step-by-step-guide/

Dann am besten die Netzwerke löschen und neu anlegen, dann haben die bereits die Version 1.0. 

```
$ p network ls
NETWORK ID    NAME        VERSION     PLUGINS
fcfcbba4e322  pnet        1.0.0       bridge,portmap,firewall,tuning
0adb872b23fd  pub1_nw     1.0.0       bridge,portmap,firewall,tuning
eb502fa10d0f  ring1_nw    1.0.0       bridge,portmap,firewall,tuning
4dcfd3cae2a9  ring2_nw    1.0.0       bridge,portmap,firewall,tuning
```


Auf dem Host: 

Ubuntu: apt install open-iscsi
modprobe iscsi_tcp 



```
# podman create -t -i \
  --hostname pcm1 \
  --shm-size 1G \
  --volume /dev/shm \
  --dns-search=example.com \
  --privileged=false  \
  --memory 1G \
  --memory-swap 1G \
  --cap-add=SYS_NICE \
  --cap-add=SYS_RESOURCE \
  --cap-add=NET_ADMIN \
  --cap-add=NET_RAW \
  --cap-add=AUDIT_WRITE \
  --cap-add=AUDIT_CONTROL \
  --restart=always \
  --systemd=true \
  --oom-kill-disable \
  --name pcm1 \
quay.io/fedora/fedora-bootc:latest
```

## Create Devices for iscsi

https://fedoraproject.org/wiki/Scsi-target-utils_Quickstart_Guide

  dnf install scsi-target-utils
  truncate -s 100M /var/tmp/iscsi-disk1
  systemctl enable --now tgtd

  tgtadm --lld iscsi --mode target --op new --tid=1 --targetname iqn.2025-01.com.example:for.all
  tgtadm --lld iscsi --mode logicalunit --op new --tid 1 --lun 1 -b /var/tmp/iscsi-disk1
  tgtadm --lld iscsi --mode target --op bind --tid 1 -I ALL
  tgtadm --lld iscsi --mode target --op show


# Client

https://www.server-world.info/en/note?os=Fedora_40&p=iscsi&f=3

```
podman create -t -i \
  --hostname pcm2 \
  --shm-size 1G \
  --volume /dev/shm \
  --dns-search=example.com \
  --privileged=false  \
  --memory 1G \
  --memory-swap 1G \
  --cap-add=SYS_NICE \
  --cap-add=SYS_RESOURCE \
  --cap-add=NET_ADMIN \
  --cap-add=NET_RAW \
  --cap-add=AUDIT_WRITE \
  --cap-add=AUDIT_CONTROL \
  --restart=always \
  --systemd=true \
  --oom-kill-disable \
  --name pcm2 \
quay.io/fedora/fedora-bootc:latest
```

dnf -y install iscsi-initiator-utils
iscsiadm -m discovery -t sendtargets -p \<ip\>

Problem bei mounten von sendtarget. 

Name resolution not working when a DNS container is started : https://github.com/containers/podman/issues/18463





  --network pasta:--ipv4-only,-a,10.0.65.4,-n,24,-g,10.0.65.1,--dns-forward,10.0.65.3,-m,1500,--no-ndp,--no-dhcpv6,--no-dhcp \
registry.fedoraproject.org/fedora-iot:latest

pasta --container pcm1 --interface eth0 --ipv4-only -a 10.0.63.4 -n 24 -g 10.0.63.1 --dns-forward 10.0.63.3 -m 1500 --no-ndp --no-dhcpv6 --no-dhcp
pasta --container mycontainer --interface eth1 --your-other-parameters


Wenn der Container rootless arbeiten soll, dann muss die Gruppe erhalten bleiben. Hierfür gibt es group-add keep-groups, dies funktioniert aber nur mit crun (ich habe 
aber das aktuelle pdoman nur mit runc compilieren können :-( Das muss man noch einmal untersuchen.)
  --group-add keep-groups \
https://github.com/containers/podman/issues/10166


```
podman network connect pub_nw --ip 10.0.65.4 pcm1
podman network connect pcm_nw --ip 10.0.62.4 pcm1
podman network connect ring1_nw --ip 10.0.61.4  pcm1
podman network connect ring2_nw --ip 10.0.63.4 pcm1
```

podman network connect podman --ip 10.88.0.5 pcm2
podman network connect pub_nw --ip 10.0.65.5 pcm2
podman network connect pcm_nw --ip 10.0.62.5 pcm2
podman network connect ring1_nw --ip 10.0.61.5  pcm2
podman network connect ring2_nw --ip 10.0.63.5 pcm2

pasta --network pub1_nw --ipv4-only --gateway 10.0.62.1 --dns 10.0.62.3 --ipv4-only -a 10.0.62.4
