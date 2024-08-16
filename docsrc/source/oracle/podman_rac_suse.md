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
unqualified-search-registries = ["container-registry.oracle.com", "registry-azure.susecloud.net", "registry.suse.com",]
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
 --env ASM_STORAGE_SIZE_GB=0 \
 localhost/oracle/rac-storage-server:latest
 ```

`podman exec racnode-storage tail -f /tmp/storage_setup.log`


### Disks anlegen

Drei OCR/Votingfiles

dd if=/dev/zero of=/oradata/asm_ocr1.img bs=1G count=1
dd if=/dev/zero of=/oradata/asm_ocr2.img bs=1G count=1
dd if=/dev/zero of=/oradata/asm_ocr3.img bs=1G count=1

Arch / olog / mlog 
dd if=/dev/zero of=/oradata/asm_arch1.img bs=1G count=8
dd if=/dev/zero of=/oradata/asm_olog1.img bs=1G count=8
dd if=/dev/zero of=/oradata/asm_mlog1.img bs=1G count=8

Data 
dd if=/dev/zero of=/oradata/asm_data1.img bs=1G count=20
dd if=/dev/zero of=/oradata/asm_data2.img bs=1G count=20
dd if=/dev/zero of=/oradata/asm_date3.img bs=1G count=20

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



