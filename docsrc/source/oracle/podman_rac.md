# RAC on Podman

https://docs.oracle.com/en/database/oracle/oracle-database/19/racpd/host-preparation-oracle-rac-podman.html

https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-AboutPodmanBuildahandSkopeo.html#podman-about

## Requirements

* Podman > 4.0.2
* Oracle Linux 8 Container image (oraclelinux:8)

## VM
https://docs.oracle.com/en/database/oracle/oracle-database/19/racpd/target-configuration-oracle-rac-podman.htm
Ziel: zwei Container auf einem Podman-Host

32 GB RAM

Disklayout
80 GB OS
SWAP 16 GB
scratch1 80 GB
scratch2 80 GB
containers 100 GB xfs
sdd-g je 50 GB

## OS Konfiguration

### Proxy

```
vi /etc/profile.d/zzz-proxy.sh

export https_proxy=http://<ip>:8080
export http_proxy=http://<ip>:8080
```

### Additional Disks

vg_container auf der 100 GB Disk erstellen und mounten

```
# pvcreate /dev/sde
# vgcreate vg_containers /dev/sde
# lvcreate -n lv_containers -l +100%FREE vg_container
# mkfs.xfs /dev/vg_containers/lv_containers

# mkdir /var/lib/containers
# vi /etc/fstab
/dev/vg_container/lv_containers  /var/lib/containers      xfs     inode64         0  2
# mount /var/lib/containers
```



https://github.com/oracle/docker-images/blob/main/OracleDatabase/RAC/OracleRealApplicationClusters/README.md#section-1--prerequisites-for-running-oracle-rac-in-containers

### DNS Pod

Nur zweck Proof-of-concept. Später überlegen, ob man das im DNS einträgt
DNS Pod: https://github.com/oracle/docker-images/blob/main/OracleDatabase/RAC/OracleDNSServer/README.md

Einiges ist bei Oracle nur nach Anmeldung möglich. Wenn aus einem github Repo dann ein podman build aufgerufen wird, kann dies scheitern. 
Daher `podman login container-registry.oracle.com`

Beim Buildscript ist ein Fehler erschienen:

```
./buildContainerImage.sh -v latest
...
Error: unknown flag: --build-arg http_proxy
```

Hier werden die --build-arg Parameter in einfachen Anführungsstrichen übergeben, das kann podman-build nicht verarbeiten. Im Script die doppelten Anführungsstriche angepasst (also gelöscht)

```
# Proxy settings
if [ -n "${http_proxy-}" ]; then
  PROXY_SETTINGS+=(--build-arg http_proxy=${http_proxy})
fi

if [ -n "${https_proxy-}" ]; then
  PROXY_SETTINGS+=(--build-arg https_proxy=${https_proxy})
fi
```


## Install

```
zypper in podman
zypper in podman-docker
zypper in buildah
zypper in skopeo
```

## podman Konfiguration

siehe podman-Doku
    signierte Images
    /etc/containers/registry.conf erweitern
        unqualified-search-registries = ["container-registry.oracle.com", ....

## sysctl settings on host

```
vi /etc/sysctl.d/98-rac.conf

fs.aio-max-nr = 1048576
fs.file-max = 6815744
net.core.rmem_max = 4194304
net.core.rmem_default = 262144
net.core.wmem_max = 1048576
net.core.wmem_default = 262144
net.core.rmem_default = 262144
```

## Create Mount Points for the Oracle Software Binaries

Vorgesehen sind dafür 2 x 80 GB Disks

```
# pvcreate /dev/sdc /dev/sdd
# vgcreate vg_scratch1 /dev/sdc
# vgcreate vg_scratch2 /dev/sdd
# lvcreate -n lv_scratch1 -l +100%FREE vg_scratch1
# lvcreate -n lv_scratch2 -l +100%FREE vg_scratch2
# mkfs.ext4 /dev/vg_scratch1/lv_scratch1
# mkfs.ext4 /dev/vg_scratch2/lv_scratch2

# vi /etc/fstab
    /dev/vg_scratch1/lv_scratch1    /scratch/rac/cluster01/node1       ext4  acl,user_xattr  0  2
    /dev/vg_scratch2/lv_scratch2    /scratch/rac/cluster01/node2       ext4  acl,user_xattr  0  2

# mkdir -p /scratch/rac/cluster01/node1
# mkdir -p /scratch/rac/cluster01/node2
# mount /scratch/rac/cluster01/node1
# mount /scratch/rac/cluster01/node2

```

## Check Shared Memory File System Mount

```
# df -h /dev/shm
Filesystem      Size  Used Avail Use% Mounted on
tmpfs            16G  180K   16G   1% /dev/shm
```

## Enable Real Time Mode for Oracle RAC Processes (geht bei SUSE mit Standard nicht, alternative?)

To run processes inside a container in real time mode for UEKR6, the /sys/fs/cgroup/cpu,cpuacct/machine.slice/cpu.rt_runtime_us must be populated with Real Time (RT) budgeting.

```
vi /etc/systemd/system/podman-rac-cgroup.service

[Unit]
Description=Populate Cgroups with real time chunk on machine restart
After=multi-user.target
[Service]
Type=oneshot
ExecStart=/bin/bash -c "echo 950000 > /sys/fs/cgroup/cpu,cpuacct/machine.slice/cpu.rt_runtime_us && \
/bin/systemctl restart podman-restart.service"
StandardOutput=journal
CPUAccounting=yes
Slice=machine.slice
[Install]
WantedBy=multi-user.target
```

```
# systemctl daemon-reload
# systemctl enable podman-restart.service
# systemctl enable podman-rac-cgroup.service --now
```


## Configure NTP on the Podman Host

chrony sollte automatisch bei sles15.5 installiert und konfiguriert sein. 

## Set Clock Source on the Podman Host

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

## Create a Directory to Stage the Oracle Software on the Podman Host

```
# mkdir -p /scratch/software/stage
# chmod 777 /scratch/software/stage
```

## runing clufy

create group: `groupadd -g 500 oinstall`

Create User: `useradd -m -d /home/oracle -s /usr/bin/bash -c "Oracle Administrator" -u 500 -g 500 oracle`

Download clufy Stand-alone vor Container Version from MOS! (Patch MOS: 30839369) 

```
unzip cvupack_host.zip
cd bin
./cluvfy comp containerhost -type podman -method root
```


## Build the Podman Image for Oracle RAC on Podman

### Prepare Container Setup Script

Create the Podman Image Build Directory

```
# mkdir /scratch/image
# cd /scratch/image
```

Create hostfile (statische IP ohne dns)

```
vi hostfile

127.0.0.1 localhost
::1 localhost ip6-localhost ip6-loopback

## Public IP addresses
10.0.20.150 racnode1.example.info racnode1
10.0.20.151 racnode2.example.info racnode2

## Virtual IP addresses
10.0.20.160 racnode1-vip.example.info racnode1-vip
10.0.20.161 racnode2-vip.example.info racnode2-vip

## Private IPs
192.168.17.150 racnode1-priv.example.info racnode1-priv
192.168.17.151 racnode2-priv.example.info racnode2-priv
```

create setupContainerEnv.sh

```
#!/bin/bash
chown grid:asmadmin /dev/asm-disk1
chown grid:asmadmin /dev/asm-disk2
chmod 660 /dev/asm-disk1
chmod 660 /dev/asm-disk2
ip route del default
# In the ip route command, replace with appropriate gateway IP
ip route add default via 10.0.20.1
cat /opt/scripts/startup/resolv.conf > /etc/resolv.conf
cat /opt/scripts/startup/hostfile > /etc/hosts
systemctl reset-failed
```

### Create a Containerfile for Oracle RAC on Podman Image

vi Containerfile

```
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
VOLUME ["/stage/software"]
VOLUME ["/u01"]
CMD ["/usr/sbin/init"]
# End of the Containerfile
```

### Create the Oracle RAC on Podman Image

```
# cd /scratch/image

# export NO_PROXY=localhost-domain 
# export http_proxy=http-proxy
# export https_proxy=https-proxy
# export version=19.22
# podman build --force-rm=true --no-cache=true --build-arg \
http_proxy=${http_proxy} --build-arg https_proxy=${https_proxy} \
-t oracle/database-rac:$version-slim  -f Containerfile .
```

```
# podman images
REPOSITORY                                    TAG         IMAGE ID      CREATED         SIZE
localhost/oracle/database-rac                 19.22-slim  20fc1f911871  30 seconds ago  509 MB
...
```

Optional, da hier auf dem podman host erzeugt und ich nur einen habe nicht notwendig. 
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

## Provision Shared Devices for Oracle ASM

2 x 50 GB je Pod für ASM Disks

```
# for i in f g h i; do echo dd if=/dev/zero of=/dev/sd$i bs=1024k count=1024; done

Execute files
```

## Create Public and Private Networks for Oracle RAC on Podman

Zwischen zwei Hosts

```
# podman network create -d bridge --subnet=10.0.20.0/24 --gateway=10.0.20.1 -o parent=ens33 rac_eth0pub1_nw
# podman network create -d bridge --subnet=192.168.17.0/24 -o parent=ens35 rac_eth1priv1_nw
# podman network create -d bridge --subnet=192.168.18.0/24 -o parent=ens36 rac_eth2priv2_nw
```

Ausnahme: container laufen auf dem selben Host
```
podman network create --subnet 192.5.0.0/16 newnet
# podman network create -d bridge --subnet=10.0.20.0/24 --gateway=10.0.20.1 rac_eth0pub1_nw
# podman network create -d bridge --subnet=192.168.17.0/24 rac_eth1priv1_nw
# podman network create -d bridge --subnet=192.168.18.0/24 rac_eth2priv2_nw


# podman network ls
NETWORK ID    NAME              DRIVER
2f259bab93aa  podman            bridge
65f93e2266e6  rac_eth0pub1_nw   bridge
c41d547c3401  rac_eth1priv1_nw  bridge
c83d993ca85b  rac_eth2priv2_nw  bridge
```
## create pod

--cpu-rt-runtime=95000  entfernt
--dns-search entfernt
--device angepasst
--memory-swap angepasst

NODE 1:

```
# podman create -t -i \
  --hostname racnode1 \
  --shm-size 2G \
  --dns-search=example.info \
  --device=/dev/sdf:/dev/asm-disk1:rw  \
  --device=/dev/sdg:/dev/asm-disk2:rw  \
  --privileged=false  \
  --volume /scratch/software/stage:/software/stage \
  --volume /scratch/rac/cluster01/node1:/u01 \
  --cpuset-cpus 0-3 \
  --memory 16G \
  --memory-swap 32G \
  --sysctl kernel.shmall=2097152  \
  --sysctl "kernel.sem=250 32000 100 128" \
  --sysctl kernel.shmmax=8589934592  \
  --sysctl kernel.shmmni=4096 \
  --sysctl 'net.ipv4.conf.eth1.rp_filter=2' \
  --sysctl 'net.ipv4.conf.eth2.rp_filter=2' \
  --cap-add=SYS_NICE \
  --cap-add=SYS_RESOURCE \
  --cap-add=NET_ADMIN \
  --cap-add=AUDIT_WRITE \
  --cap-add=AUDIT_CONTROL \
  --restart=always \
  --ulimit rtprio=99  \
  --systemd=true \
  --name racnode1 \
oracle/database-rac:19.22-slim
```

NODE 2:

```
# podman create -t -i \
  --hostname racnode2 \
  --shm-size 2G \
  --dns-search=example.info \
  --device=/dev/sdh:/dev/asm-disk1:rw  \
  --device=/dev/sdi:/dev/asm-disk2:rw  \
  --privileged=false  \
  --volume /scratch/rac/cluster01/node2:/u01 \
  --cpuset-cpus 0-3 \
  --memory 16G \
  --memory-swap 32G \
  --sysctl kernel.shmall=2097152  \
  --sysctl "kernel.sem=250 32000 100 128" \
  --sysctl kernel.shmmax=8589934592  \
  --sysctl kernel.shmmni=4096 \
  --sysctl 'net.ipv4.conf.eth1.rp_filter=2' \
  --sysctl 'net.ipv4.conf.eth2.rp_filter=2' \
  --cap-add=SYS_NICE \
  --cap-add=SYS_RESOURCE \
  --cap-add=NET_ADMIN \
  --cap-add=AUDIT_WRITE \
  --cap-add=AUDIT_CONTROL \
  --restart=always \
  --ulimit rtprio=99  \
  --name racnode2 \
oracle/database-rac:19.22-slim
```

## Assign Networks to the Oracle RAC Containers

Use these procedures to assign networks to each of the Oracle Real Application Clusters (Oracle RAC) nodes that you create in the Oracle RAC on Podman containers.

To ensure that the network interface name used by each node for a given network is the same, each node must use the exact same order of the disconnect and connect commands to the associated networks. For example, consistently across all nodes, the eth0 interface is public. The eth1 interface is the first private network interface, and the eth2 interface is the second private network interface.

NODE 1:

```
# podman network disconnect podman racnode1
# podman network connect rac_eth0pub1_nw --ip 10.0.20.150 racnode1
# podman network connect rac_eth1priv1_nw --ip 192.168.17.150  racnode1
# podman network connect rac_eth2priv2_nw --ip 192.168.18.150  racnode1
```

NODE 2:

```
# podman network disconnect podman racnode2
# podman network connect rac_eth0pub1_nw --ip 10.0.20.151 racnode2
# podman network connect rac_eth1priv1_nw --ip 192.168.17.151  racnode2
# podman network connect rac_eth2priv2_nw --ip 192.168.18.151  racnode2
```

## Start the Podman Containers and Connect to the Network

NODE1:

```
# systemctl start podman-rac-cgroup.service
# podman start racnode1
```

NODE2:
```
# systemctl start podman-rac-cgroup.service
# podman start racnode2
```

## Adjust Memlock Limits

To ensure that the container total memory is included in calculating host memlock limit, adjust the limit in containers after Podman containers are created.

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

## Download Oracle Grid Infrastructure and Oracle Database Software

und ablegen in /scratch/software/stage

# Install GRID

https://docs.oracle.com/en/database/oracle/oracle-database/19/racpd/example-installing-oracle-grid-infrastructure-and-oracle-rac-podman.html#GUID-B4949A10-6FEB-4AE5-90BF-4DFEE410A068

## Create Paths and Change Permissions

NODE 1:

```
# podman exec racnode1 /bin/bash -c "mkdir -p /u01/app/oraInventory"
# podman exec racnode1 /bin/bash -c "mkdir -p /u01/app/grid"
# podman exec racnode1 /bin/bash -c "mkdir -p /u01/app/19c/grid"
# podman exec racnode1 /bin/bash -c "chown -R grid:oinstall /u01/app/grid"
# podman exec racnode1 /bin/bash -c "chown -R grid:oinstall /u01/app/19c/grid"
# podman exec racnode1 /bin/bash -c "chown -R grid:oinstall /u01/app/oraInventory"
# podman exec racnode1 /bin/bash -c "mkdir -p /u01/app/oracle"
# podman exec racnode1 /bin/bash -c "mkdir -p /u01/app/oracle/product/19c/dbhome_1"
# podman exec racnode1 /bin/bash -c "chown -R oracle:oinstall /u01/app/oracle"
# podman exec racnode1 /bin/bash -c "chown -R oracle:oinstall /u01/app/oracle/product/19c/dbhome_1"
```

NODE 2:

``` 
# podman exec racnode2 /bin/bash -c "mkdir -p /u01/app/oraInventory"
# podman exec racnode2 /bin/bash -c "mkdir -p /u01/app/grid"
# podman exec racnode2 /bin/bash -c "mkdir -p /u01/app/19c/grid"
# podman exec racnode2 /bin/bash -c "chown -R grid:oinstall /u01/app/grid"
# podman exec racnode2 /bin/bash -c "chown -R grid:oinstall /u01/app/19c/grid"
# podman exec racnode2 /bin/bash -c "chown -R grid:oinstall /u01/app/oraInventory"
# podman exec racnode2 /bin/bash -c "mkdir -p /u01/app/oracle"
# podman exec racnode2 /bin/bash -c "mkdir -p /u01/app/oracle/product/19c/dbhome_1"
# podman exec racnode2 /bin/bash -c "chown -R oracle:oinstall /u01/app/oracle"
# podman exec racnode2 /bin/bash -c "chown -R oracle:oinstall /u01/app/oracle/product/19c/dbhome_1"
```