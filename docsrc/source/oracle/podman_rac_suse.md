# RAC on SUSE Podman 

https://docs.oracle.com/en/database/oracle/oracle-database/19/racpd/host-preparation-oracle-rac-podman.html
https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-AboutPodmanBuildahandSkopeo.html#podman-about

## Disklayout

scratch     128 GB  Image on other stuff
container   128 GB  containerfiles
asm-disks   256 GB  RAC ASM Diskcontainer


## Disks erstellen

### Scratch

```
pvcreate /dev/sdc
vgcreate vg_scratch /dev/sdc
lvcreate -n lv_scratch -l +100%FREE vg_scratch
mkfs.ext4 /dev/vg_scratch/lv_scratch
echo "/dev/vg_scratch1/lv_scratch1    /scratch/rac/cluster01/node1       ext4  acl,user_xattr  0  2" >> /etc/fstab
mkdir -p /scratch
mount /scratch
```

### containter files

```
pvcreate /dev/sdd
vgcreate vg_containers /dev/sdd
lvcreate -n lv_containers -l +100%FREE vg_containers
mkfs.xfs /dev/vg_containers/lv_containers
echo "/dev/vg_containers/lv_containers  /var/lib/containers      xfs     inode64         0  2" >> /etc/fstab
mkdir -p /var/lib/containers
mount /var/lib/containers
```

### asm files

```
pvcreate /dev/sde
vgcreate vg_oradata /dev/sde
lvcreate -n lv_oradata -l +100%FREE vg_oradata
mkfs.xfs /dev/vg_oradata/lv_oradata
echo "/dev/vg_oradata/lv_oradata  /oradata      xfs     inode64         0  2" >> /etc/fstab
mkdir -p /oradata
mount /oradata
```

```
df -h 
...
/dev/mapper/vg_scratch-lv_scratch        126G   24K  120G   1% /scratch
/dev/mapper/vg_containers-lv_containers  128G  163M  128G   1% /var/lib/containers
/dev/mapper/vg_oradata-lv_oradata        256G  294M  256G   1% /oradata
```

## sysctl settings

vi /etc/sysctl.d/98-rac.conf

```
fs.aio-max-nr = 1048576
fs.file-max = 6815744
net.core.rmem_max = 4194304
net.core.rmem_default = 262144
net.core.wmem_max = 1048576
net.core.wmem_default = 262144
net.core.rmem_default = 262144
```

## sonstige Software 

`sudo zypper in yast2-nfs-server` Wird vermutlich benötigt für den nfs-container von Oracle

`sudo zypper in git`    braucht man immer


## Umgebung

`vi ~/.bashrc`

```
pe() {
  podman exec -ti $1 /bin/bash
}

alias p=podman
```

`source ~/.bashrc`


## Configure NTP on the Podman Host

chrony sollte automatisch bei sles15.5 installiert und konfiguriert sein. 

### Set Clock Source on the Podman Host

Oracle recommends that you set the clock source to tsc for better performance in virtual environments (VM) on Linux x86-64.

```
cat /sys/devices/system/clocksource/clocksource0/available_clocksource

cat /sys/devices/system/clocksource/clocksource0/current_clocksource
tsc

ansonsten

# echo "tsc">/sys/devices/system/clocksource/clocksource0/current_clocksource

und GRUB anpassen mit clocksource=tsc
# GRUB_CMDLINE_LINUX="rd.lvm.lv=ol/root rd.lvm.lv=ol/swap rhgb quiet numa=off transparent_hugepage=never clocksource=tsc" 
```

## sapcd Laufwerk einbinden

Gemäß Prozedur über azure



## Installation von Podman 

```
zypper in podman
zypper in podman-docker
zypper in buildah
zypper in skopeo
```

## Configuration von Podman

### /etc/containers/registries.conf

```
unqualified-search-registries = ["container-registry.oracle.com", "registry.opensuse.org", "registry.suse.com", "docker.io"]
```

### signed Images von Oracle

1. Container Registries anpassen

```
vi /etc/containers/registries.d/oracle.yaml

docker:
  container-registry.oracle.com:
    sigstore: https://container-trust.oci.oraclecloud.com/podman
```

2. Download des public GPG keys 

```
mkdir -p /etc/pki/containers
wget -O /etc/pki/containers/GPG-KEY-oracle https://container-trust.oci.oraclecloud.com/podman/GPG-KEY-oracle
```

3. Anpassen der Container policy configuration


```
{
  "default": [
    {
      "type": "insecureAcceptAnything"
    }
  ],
  "transports":
    {
      "docker-daemon":
        {
          "": [{"type":"insecureAcceptAnything"}]
        },
      "docker":
        {
          "container-registry.oracle.com": [
            {
              "type": "signedBy",
              "keyType": "GPGKeys",
              "keyPath": "/etc/pki/containers/GPG-KEY-oracle"
            }
          ]
        }
    }
}

```

4. Testen
Hier muss man einmal den keyPath austauschen in der policy.json, damit man einen Fehler erzeugt, dass der Key nicht funktioniert. Bei richtiger Konfiguration unterscheidet sich das ansonsten nicht von dem normalen podman pull ohne Signatur

```
podman pull oraclelinux:8

Resolved "oraclelinux" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull container-registry.oracle.com/os/oraclelinux:8...
Error: copying system image from manifest list: Source image rejected: No public keys imported
```


# Oracle Container

## Oracle Docker-Images kopieren

`git clone https://github.com/oracle/docker-images.git`

## Netzwerkbriges erzeugen

Da der RAC auf einem Host betrieben wird, können hier bridges verwendet werden. Ansonsten soll man wohl MACVLAN verwenden.


```
podman network create --driver=bridge --subnet=172.16.1.0/24 rac_pub1_nw
podman network create --driver=bridge --subnet=192.168.17.0/24 rac_priv1_nw
podman network create --driver=bridge --subnet=192.168.18.0/24 rac_priv2_nw
```

```
podman network ls

NETWORK ID    NAME          DRIVER
2f259bab93aa  podman        bridge
ce28ac547615  rac_priv1_nw  bridge
1d0070aceddb  rac_priv2_nw  bridge
a72bb6bf9c50  rac_pub1_nw   bridge
``` 

## Oracle DNS Server

Oracle bietet unter https://github.com/oracle/docker-images/tree/main/OracleDatabase/RAC/OracleDNSServer einen DNS Server an, der hier verwendet wird.

### Create Zonefile & Reversezonefile

```
cd ~/docker-images/OracleDatabase/RAC/OracleDNSServer/dockerfiles/latest
mv zonefile zonefile.bak
mv reversezonefile reversezonefile.bak
```

vi zonefile
```
$TTL 86400
@       IN SOA  example.com.   root (
        2014090401    ; serial
        3600    ; refresh
        1800    ; retry
        604800    ; expire
        86400 )  ; minimum
; Name server's
                IN NS      example.com.
                IN A       172.16.1.25
; Name server hostname to IP resolve.
;@ORIGIN  eot.us.oracle.com.
;@               IN      NS      gns.eot.us.oracle.com.
; Hosts in this Domain
racdns                               IN A    172.16.1.25
racnode1                             IN A    172.16.1.150
racnode2                             IN A    172.16.1.151
racnode1-vip                         IN A    172.16.1.160
racnode2-vip                         IN A    172.16.1.161
racnode1-priv1                       IN A    192.168.17.150
racnode2-priv1                       IN A    192.168.17.151
racnode1-priv2                       IN A    192.168.18.150
racnode2-priv2                       IN A    192.168.18.151
racnode-scan                         IN A    172.16.1.170
racnode-scan                         IN A    172.16.1.171
racnode-scan                         IN A    172.16.1.172
racnode-gns1                         IN A    172.16.1.175
racnode-gns2                         IN A    172.16.1.176

; CMAN Server Entry
;racnode-cman1         IN A    172.16.1.2
racnode-cman2         IN A    172.16.1.180
racnode-cman3         IN A    172.16.1.181
racnode-cman4         IN A    172.16.1.182
```

vi reversezonefile
```
$ORIGIN 1.16.172.in-addr.arpa.
$TTL 86400
@       IN SOA  racdns.example.com. root.example.com. (
        2014090402      ; serial
        3600      ; refresh
        1800      ; retry
        604800      ; expire
        86400 )    ; minimum
; Name server's
        IN      NS     racnode-dns.example.com.
; Name server hostname to IP resolve.
25      IN PTR  racdns.example.com.
; Second RAC Cluster on Same Subnet on Docker
150     IN PTR  racnode1.example.com.
151     IN PTR  racnode2.example.com.
151     IN PTR  racnode2.example.com.
160     IN PTR  racnode1-vip.example.com.
161     IN PTR  racnode2-vip.example.com.

; SCAN IPs
170     IN PTR  racnode-scan.example.com.
171     IN PTR  racnode-scan.example.com.
172     IN PTR  racnode-scan.example.com.

;GNS
175     IN PTR  racnode-gns1.example.com.
176     IN PTR  racnode-gns2.example.com.

; CMAN Server Entry
180       IN PTR  racnode-cman2.example.com.
181       IN PTR  racnode-cman3.example.com.
182       IN PTR  racnode-cman4.example.com.
```

Nur zweck Proof-of-concept. Später überlegen, ob man das im DNS einträgt
DNS Pod: https://github.com/oracle/docker-images/blob/main/OracleDatabase/RAC/OracleDNSServer/README.md

Einiges ist bei Oracle nur nach Anmeldung möglich. Wenn aus einem github Repo dann ein podman build aufgerufen wird, kann dies scheitern. 
Daher `podman login container-registry.oracle.com`

### run buildscript

```
cd ~/docker-images/OracleDatabase/RAC/OracleDNSServer/dockerfiles/lates
./buildContainerImage.sh -v latest
```

### create & start DNS container

```
podman create --hostname racdns \
  --dns-search=example.com \
  --cap-add=AUDIT_WRITE \
  -e DOMAIN_NAME="example.com" \
  -e WEBMIN_ENABLED=false \
  -e RAC_NODE_NAME_PREFIXP="racnodep" \
  -e SETUP_DNS_CONFIG_FILES="setup_true"  \
  --privileged=false \
  --name racdns \
 oracle/rac-dnsserver:latest
```

```
podman network disconnect podman racdns
podman network connect rac_pub1_nw --ip 172.16.1.25 racdns
podman network connect rac_priv1_nw --ip 192.168.17.25 racdns
podman start racdns
```

`podman logs -f racdns`

## storage container

https://github.com/oracle/docker-images/blob/main/OracleDatabase/RAC/OracleRACStorageServer/README.md

Für die Version 0.1 werden die asm-disks vom NFS bezogen. Hierfür gibt es einen eigenen Storagecontainer von Oracle.
Im Standard werden fünf "asm" Disks erstellt. Da wir hier einen anderen Split vornehmen, wird das erst einmal unterbunden und dann manuell 
erstellt. 
Die Disks liegen im Verzeichnis /oradata.

### Containerimage erstellen

```
cd <git-cloned-path>/docker-images/OracleDatabase/RAC/OracleRACStorageServer/dockerfiles
./buildDockerImage.sh -v latest
```

### Container erstellen

Wir wollen ohne ASM Disks starten, daher ASM_STORAGE_SIZE=0, ansonsten werden 5 Disks erstellt nach der Formel ASM_STORAGE_SIZE / 5


```
podman run -d -t \
 --hostname racnode-storage \
 --dns-search=example.com  \
 --cap-add SYS_ADMIN \
 --cap-add AUDIT_WRITE \
 --cap-add NET_ADMIN \
 --volume /oradata/:/oradata \
 --network=rac_priv1_nw \
 --ip=192.168.17.80 \
 --systemd=always \
 --restart=always \
 --name racnode-storage \
 --privileged=true \
 --env ASM_STORAGE_SIZE_GB=0 \
 localhost/oracle/rac-storage-server:latest
 ```

`podman exec racnode-storage tail -f /tmp/storage_setup.log`


### Disks anlegen

Drei OCR/Votingfiles

dd if=/dev/zero of=/oradata/asm_ocr1.img bs=1G count=1
dd if=/dev/zero of=/oradata/asm_ocr2.img bs=1G count=1
dd if=/dev/zero of=/oradata/asm_ocr3.img bs=1G count=1

Arch / olog / mlog / fra
dd if=/dev/zero of=/oradata/asm_arch1.img bs=1G count=8
dd if=/dev/zero of=/oradata/asm_olog1.img bs=1G count=8
dd if=/dev/zero of=/oradata/asm_mlog1.img bs=1G count=8
dd if=/dev/zero of=/oradata/asm_fra1.img bs=1G count=16



Data 
dd if=/dev/zero of=/oradata/asm_data1.img bs=1G count=20
dd if=/dev/zero of=/oradata/asm_data2.img bs=1G count=20
dd if=/dev/zero of=/oradata/asm_data3.img bs=1G count=10


Lokalen files 
mkdir -p /oradata/node1
mkdir -p /oradata/node2
mkdir -p /oradata/sidnode1
mkdir -p /oradata/sidnode2

### NFS Volume anlegen:

```
podman volume create --driver local \
--opt type=nfs \
--opt   o=addr=192.168.17.80,rw,bg,hard,tcp,vers=3,timeo=600,rsize=32768,wsize=32768,actimeo=0 \
--opt device=192.168.17.80:/oradata \
racstorage
```

## Oracle RAC Podman Basisimage erstellen (hier nur OS)

### Vorbereitungen für das Container Setup Script 

1. Podman Image Build Directory erzeugen

```
# mkdir /scratch/image
# cd /scratch/image
```

2. hostfile (statische IP ohne dns) erstellen

```
vi hostfile

127.0.0.1 localhost
::1 localhost ip6-localhost ip6-loopback

## Public IP addresses
172.16.1.150 racnode1.example.com racnode1
172.16.1.151 racnode2.example.com racnode2

## Virtual IP addresses
172.16.1.160 racnode1-vip.example.com racnode1-vip
172.16.1.161 racnode2-vip.example.com racnode2-vip

## Private IPs
192.168.17.150 racnode1-priv1.example.com racnode1-priv1
192.168.17.151 racnode2-priv1.example.com racnode2-priv1
192.168.18.150 racnode1-priv2.example.com racnode1-priv2
192.168.18.151 racnode2-priv2.example.com racnode2-priv2

172.16.1.170 racnode-scan.example.com racnode-scan
172.16.1.171 racnode-scan.example.com racnode-scan
172.16.1.172 racnode-scan.example.com racnode-scan
```

3. resolv.conf erstellen

```
vi resolv.conf

### /etc/resolv.conf is a symlink to /run/netconfig/resolv.conf
### autogenerated by netconfig!
#
# Before you change this file manually, consider to define the
# static DNS configuration using the following variables in the
# /etc/sysconfig/network/config file:
#     NETCONFIG_DNS_STATIC_SEARCHLIST
#     NETCONFIG_DNS_STATIC_SERVERS
#     NETCONFIG_DNS_FORWARDER
# or disable DNS configuration updates via netconfig by setting:
#     NETCONFIG_DNS_POLICY=''
#
# See also the netconfig(8) manual page and other documentation.
#
### Call "netconfig update -f" to force adjusting of /etc/resolv.conf.
search example.com
nameserver 172.16.1.25
```

4. setupContainerEnv.sh erzeugen

```
vi setupContainerEnv.sh


#!/bin/bash
# andere Variante
# chown grid:asmadmin /dev/asm-disk1
# chown grid:asmadmin /dev/asm-disk2
# chmod 660 /dev/asm-disk1
# chmod 660 /dev/asm-disk2
ip route del default
# In the ip route command, replace with appropriate gateway IP
ip route add default via 172.16.1.1
cat /opt/scripts/startup/resolv.conf > /etc/resolv.conf
cat /opt/scripts/startup/hostfile > /etc/hosts
systemctl reset-failed
```


5. Containerfile für Oracle RAC Podman Image erzeugen



```
vi Containerfile


# Pull base image
# ---------------
FROM oraclelinux:8
# Environment variables required for this build (do NOT change)
# -------------------------------------------------------------
## Environment Variables
## ---
ENV SCRIPT_DIR=/opt/scripts/startup \
    RESOLVCONFENV="resolv.conf" \
    HOSTFILEENV="hostfile" \
    SETUPCONTAINERENV="setupContainerEnv.sh"

### Copy Files
# ----

COPY  $SETUPCONTAINERENV $SCRIPT_DIR/

### RUN Commands
# -----
COPY $HOSTFILEENV $RESOLVCONFENV $SCRIPT_DIR/
RUN dnf install -y oracle-database-preinstall-19c systemd vim passwd openssh-server hostname xterm xhost vi policycoreutils-python-utils && \
 dnf clean all && \
 sync && \
 groupadd -g 54334 asmadmin && \
 groupadd -g 54335 asmdba && \
 groupadd -g 54336 asmoper && \
 useradd -u 54332 -g oinstall -G oinstall,asmadmin,asmdba,asmoper,racdba,dba grid && \
 usermod -g oinstall -G oinstall,dba,oper,backupdba,dgdba,kmdba,asmdba,racdba,asmadmin oracle && \
 cp /etc/security/limits.d/oracle-database-preinstall-19c.conf /etc/security/limits.d/grid-database-preinstall-19c.conf && \
 sed -i 's/oracle/grid/g' /etc/security/limits.d/grid-database-preinstall-19c.conf && \
 rm -f /etc/rc.d/init.d/oracle-database-preinstall-19c-firstboot && \
 rm -f /etc/sysctl.conf && \
 rm -f /usr/lib/systemd/system/dnf-makecache.service && \
 echo "$SCRIPT_DIR/$SETUPCONTAINERENV" >> /etc/rc.local && \
 chmod +x $SCRIPT_DIR/$SETUPCONTAINERENV && \
 chmod +x /etc/rc.d/rc.local && \
 setcap 'cap_net_admin,cap_net_raw+ep' /usr/bin/ping && \
 sync

USER root
WORKDIR /root
VOLUME ["/oradata"]
VOLUME ["/oracle"]
CMD ["/usr/sbin/init"]
# End of the Containerfile
```


### Oracle RAC Podman Basisimage erstellen

```
# cd /scratch/image

# export NO_PROXY=localhost-domain 
# export http_proxy=http-proxy
# export https_proxy=https-proxy
# export version=19.22
# podman build --force-rm=true --no-cache=true --build-arg http_proxy=${http_proxy} --build-arg https_proxy=${https_proxy} -t oracle/database-rac:$version-slim  -f Containerfile .
```

Images anzeigen: 

```
# podman images
REPOSITORY                                    TAG         IMAGE ID      CREATED         SIZE
localhost/oracle/database-rac                 19.22-slim  20fc1f911871  30 seconds ago  509 MB
...
```

Optional, da hier auf dem podman host erzeugt und auch nur ein Server vorhanden nicht notwendig, ansonsten 
Sichern des Images und Kopie auf den Podmanhost:
```
# podman image save -o /var/tmp/database-rac.tar localhost/oracle/database-rac:19.16-slim 
# scp -i <ssh key for host podman-host-2> /var/tmp/database-rac.tar opc@podman-host-2:/var/tmp/database-rac.tar

Auf dem anderen Podman-Host:
# podman image load -i /var/tmp/database-rac.tar 
# podman images
REPOSITORY                                    TAG         IMAGE ID      CREATED        SIZE
localhost/oracle/database-rac                 19.16-slim  9d5fed7eb7ba  2 minutes ago  396 MB
container-registry.oracle.com/os/oraclelinux  8           dfce5863ff0f  2 months ago   243 MB
```

## Oracle RAC Basisimage starten

### Container erzeugen

*Hier noch überlegen, wie mit dem DB SID Verzeichnis umgegangen werden soll. Ein Container kann man im Nachhinein nicht ändern, d.h. hier müsste dann auch schon das /oracle/sid Verzeichnis angelegt werden. Forschen, ob es noch eine andere Möglichkeit gibt.* (https://github.com/containers/podman/issues/1320)

*für das Anlegen einer späteren Datenbank mzss /dev/shm gemountet werden* 

```
# podman create -t -i \
  --hostname racnode1 \
  --dns-search=example.com \
  --privileged=false  \
  --security-opt apparmor=unconfined \
  --volume /dev/shm \
  --volume racstorage:/oradata \
  --volume /oradata/node1:/oracle \
  --volume /mnt/sapcd:/software/stage \
  --memory 16G \
  --memory-swap 32G \
  --sysctl kernel.shmall=2097152  \
  --sysctl "kernel.sem=250 32000 100 128" \
  --sysctl kernel.shmmax=8589934592  \
  --sysctl kernel.shmmni=4096 \
  --sysctl 'net.ipv4.conf.eth1.rp_filter=2' \
  --sysctl 'net.ipv4.conf.eth2.rp_filter=2' \
  --sysctl "net.ipv4.ping_group_range=0 2147483647" \
  --cap-add=SYS_NICE \
  --cap-add=SYS_RESOURCE \
  --cap-add=NET_ADMIN \
  --cap-add=NET_RAW \
  --cap-add=AUDIT_WRITE \
  --cap-add=AUDIT_CONTROL \
  --cap-add CAP_SYS_ADMIN \
  --restart=always \
  --ulimit rtprio=99  \
  --systemd=true \
  --log-level=error \
  --name racnode1g \
racnode1grid

oracle/database-rac:19.22-slim
  --sysctl "vm.hugetlb_shm_group=54322" \
  --sysctl "vm.nr_hugepages=5632" \
    --tmpfs /dev/shm:rw,exec,size=4G \

# podman create -t -i \
  --hostname racnode2 \
  --dns-search=example.com \
  --privileged=false  \
  --security-opt apparmor=unconfined \
  --volume /dev/shm \
  --volume racstorage:/oradata \
  --volume /oradata/node2:/oracle \
  --volume /mnt/sapcd:/software/stage \
  --memory 16G \
  --memory-swap 32G \
  --sysctl kernel.shmall=2097152  \
  --sysctl "kernel.sem=250 32000 100 128" \
  --sysctl kernel.shmmax=8589934592  \
  --sysctl kernel.shmmni=4096 \
  --sysctl 'net.ipv4.conf.eth1.rp_filter=2' \
  --sysctl 'net.ipv4.conf.eth2.rp_filter=2' \
  --sysctl "net.ipv4.ping_group_range=0 2147483647" \
  --cap-add=SYS_NICE \
  --cap-add=SYS_RESOURCE \
  --cap-add=NET_ADMIN \
  --cap-add=NET_RAW \
  --cap-add=AUDIT_WRITE \
  --cap-add=AUDIT_CONTROL \
  --cap-add CAP_SYS_ADMIN \
  --restart=always \
  --ulimit rtprio=99  \
  --systemd=true \
  --log-level=error \
  --name racnode2g \
racnode2grid
```
oracle/database-rac:19.22-slim

### Public und Private Networks den Containern zuordnen

Da wir nur auf einem Host zwei Pods erstellen, ist hier das bridge-Network zu verwenden. Weiter oben wurden bereits die Netze definiert: 

```
# podman network ls
NETWORK ID    NAME          DRIVER
2f259bab93aa  podman        bridge
ce28ac547615  rac_priv1_nw  bridge  <-- cache fusion #1
1d0070aceddb  rac_priv2_nw  bridge  <-- cache fusion #2
a72bb6bf9c50  rac_pub1_nw   bridge  <-- public interface
```

Den Containern wird eine IP sowie das Netwerk zugewiesen: 

```
NODE 1:

podman network disconnect podman racnode1g
podman network connect rac_pub1_nw --ip 172.16.1.150 racnode1g
podman network connect rac_priv1_nw --ip 192.168.17.150  racnode1g
podman network connect rac_priv2_nw --ip 192.168.18.150  racnode1g
```

NODE 2:

```
podman network disconnect podman racnode2g
podman network connect rac_pub1_nw --ip 172.16.1.151 racnode2g
podman network connect rac_priv1_nw --ip 192.168.17.151  racnode2g
podman network connect rac_priv2_nw --ip 192.168.18.151  racnode2g

```

### Start der Podman Container

NODE1:

```
# systemctl start podman-rac-cgroup.service (geht noch nicht)
# podman start racnode1
```

NODE2:
```
# systemctl start podman-rac-cgroup.service (geht noch nicht)
# podman start racnode2
```

Prüfen der DNS Namensauflösung und Filesystemmounts mit `podman exec -ti racnode1 /bin/bash

```
[root@racnode2 ~]# df -h
Filesystem                                           Size  Used Avail Use% Mounted on
overlay                                              128G  1.3G  127G   1% /
tmpfs                                                 16G     0   16G   0% /tmp
tmpfs                                                 16G   56K   16G   1% /run
tmpfs                                                 64M     0   64M   0% /dev
/dev/mapper/vg_oradata-lv_oradata                    256G   68G  189G  27% /oracle  <-- eigenes Volume
192.168.17.80:/oradata                               256G   68G  189G  27% /oradata <-- NFS Share vom racnode-storage
tmpfs                                                6.3G  1.5M  6.3G   1% /etc/hosts
tmpfs                                                 16G     0   16G   0% /run/lock
//swdepotstorageaccount.file.core.windows.net/sapcd  512G   43G  470G   9% /software/stage   <-- Mount eines NFS Shares vom Host
tmpfs                                                4.0G     0  4.0G   0% /dev/shm
tmpfs                                                 16G     0   16G   0% /sys/fs/cgroup
tmpfs                                                 16G  8.0M   16G   1% /var/log/journal
tmpfs                                                 16G     0   16G   0% /proc/acpi
tmpfs                                                 16G     0   16G   0% /sys/firmware
tmpfs                                                 16G     0   16G   0% /sys/dev/block
```

## Anpassen der Memlock Limits

Um sicherzustellen, dass die container total memory im Host memlock limit enhalten ist, muss man das limit der container anpassen

In beiden Containern!

```
podman exec -ti racnode1 /bin/bash

CONTAINER_MEMORY=$(cat /sys/fs/cgroup/memory/memory.limit_in_bytes)
echo $CONTAINER_MEMORY
echo $((($CONTAINER_MEMORY/1024)*9/10))

grep memlock /etc/security/limits.d/oracle-database-preinstall-19c.conf

sed -i -e 's,<found value>,<new calculated value>,g' /etc/security/limits.d/oracle-database-preinstall-19c.conf

# Ebenso für die grid

grep memlock /etc/security/limits.d/grid-database-preinstall-19c.conf
sed -i -e 's,<found value>,<new calculated value>,g' /etc/security/limits.d/grid-database-preinstall-19c.conf

```

## weitere Schweinereien

https://cloud.google.com/bare-metal/docs/troubleshooting/troubleshoot-oracle-rac-issues?hl=de

CRS root.sh oder OCSSD schlägt mit Fehler No Network HB fehl
Das CRS-Skript root.sh schlägt mit dem folgenden Fehler fehl, wenn der Knoten die IP-Adresse 169.254.169.254 pingt:

`has a disk HB, but no network HB`

Die IP-Adresse 169.254.169.254 ist der Google Cloud-Metadatendienst, der die Instanz in Google Cloud registriert. Wenn Sie diese IP-Adresse blockieren, kann die Google Cloud-VM nicht gestartet werden. Dies kann wiederum die HAIP-Kommunikationsroute unterbrechen, sodass auf den RAC-Servern der Bare-Metal-Lösung HAIP-Kommunikationsprobleme auftreten.

Um dieses Problem zu beheben, müssen Sie die IP-Adresse blockieren oder HAIP deaktivieren. Das folgende Beispiel zeigt, wie IP-Adressen mit route-Befehlen blockiert werden. Die Änderungen, die von der route-Anweisung vorgenommen werden, sind nicht dauerhaft. Daher müssen Sie die Systemstartscripts ändern.

So beheben Sie das Problem:

`/sbin/route add -host 169.254.169.254 reject`

persistent: 
`chmod +x /etc/rc.d/rc.local`
Fügen Sie in der Datei /etc/rc.d/rc.local die folgenden Zeilen hinzu:
```
/sbin/route add -host 169.254.169.254 reject
```

Enable rc-local service
```
systemctl status rc-local.service
systemctl enable rc-local.service
systemctl start rc-local.service
```


# Installation der Oracle GRID

nach interner Dokumentation mit ocr votingdisk und Management-B

# Installation Oracle NonSAP 

nach interner Dokumentation

Fehler: 

/dev/shm wird benötigt, ansonsten kann kein shared memory benutzt werden. chmod 1777 auf /dev/shm setzen, damit oracle auch den Bereich nutzen kann. 


Oracle Support Document 2850137.1 (ORA-00600: internal error code, arguments: [ksipc_ksmsq_init] while creating a database using DBCA) can be found at: https://support.oracle.com/epmos/faces/DocumentDisplay?id=2850137.1

