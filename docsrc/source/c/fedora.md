# Fedora Cloud Image

https://docs.fedoraproject.org/en-US/fedora-server/virtualization/vm-install-cloudimg-centos9/

https://codeofconnor.com/a-faster-way-to-create-virtual-machines-with-cloud-images-and-virt-manager/

## Download Cloud Image

```
mkdir /var/lib/libvirt/images/base
cd /var/lib/libvirt/images/base
curl -LkO https://download.fedoraproject.org/pub/fedora/linux/releases/34/Cloud/x86_64/images/Fedora-Cloud-Base-34-1.2.x86_64.qcow2
```

## Generate a cloud-init bootstrap image

```
mkdir cloud-init-bootstrap
cd cloud-init-bootstrap
touch meta-data
cat > user-data <<EOF
#cloud-config

system_info:
  default_user:
    name: fedora

chpasswd:
  list: |
    fedora:password
  expire: False

resize_rootfs: True
ssh_authorized_keys:
   - ssh-rsa AAAAB3Nza...
EOF

genisoimage -output seedci.iso -volid cidata -joliet -rock user-data meta-data

```

## Create a disk for the new VM

```
sudo qemu-img create -f qcow2 \
  -b /var/lib/libvirt/images/base/Fedora-Cloud-Base-Generic.x86_64-40-1.14.qcow2 \
  -F qcow2 \
  /var/lib/libvirt/images/fedora-40.qcow2 8G

```

## Create the new domain (VM)

```
virt-install --name fedora-40 --os-variant fedora38 --vcpus 2 --memory 1024 \
  --graphics vnc --virt-type kvm --disk /var/lib/libvirt/images/fedora-40.qcow2 \
  --cdrom /var/lib/libvirt/images/base/cloud-init-bootstrap/seedci.iso
```

## Cloud Init

Cloud Init kann auch erweitert werden im o.g. Script z.B. durch hinzufügen des public key, bsp.:

```
#cloud-config

system_info:
   default_user:
     name: fedora

chpasswd:
  list: |
    fedora:password
  expire: False
 

resize_rootfs: True
ssh_authorized_keys:
   - ssh-rsa AAAAB3Nza...
```


# Fedora CoreOS

Container OS: https://fedoraproject.org/coreos/

## Produce Ignition File

https://docs.fedoraproject.org/en-US/fedora-coreos/producing-ign/

### Download Butane

```
# Download signed keys and import
curl https://fedoraproject.org/fedora.gpg | gpg --import

# get image
wget https://github.com/coreos/butane/releases/download/v0.21.0/butane-x86_64-unknown-linux-gnu
wget https://github.com/coreos/butane/releases/download/v0.21.0/butane-x86_64-unknown-linux-gnu.asc

# verify
gpg --verify butane-x86_64-unknown-linux-gnu.asc butane-x86_64-unknown-linux-gnu

# rename
mv butane-x86_64-unknown-linux-gnu butane
```

Datei fcos.bu erstellen und public key eintragen
```
variant: fcos
version: 1.5.0
passwd:
  users:
    - name: core
      ssh_authorized_keys:
        - ssh-rsa AAAA...
```

Erstelle ignition file
```
./butane --pretty --strict fcos.bu > fcos.ign

# zweiter Test
./butane --pretty --strict -d . fcos.bu > fcos.ign
```

## Fedora Core OS

### Download

Download ova Image (https://fedoraproject.org/coreos/download?stream=stable#baremetal)

```
wget https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/40.20240616.3.0/x86_64/fedora-coreos-40.20240616.3.0-vmware.x86_64.ova

# Checksum-File downloaden

# Signaturen downloaden

# Import Fedora gpg key
curl -O https://fedoraproject.org/fedora.gpg

# Verifizierung signatur-file 
gpgv --keyring ./fedora.gpg fedora-coreos-40.20240616.3.0-vmware.x86_64.ova.sig fedora-coreos-40.20240616.3.0-vmware.x86_64.ova

# Verifizierung checksumme
sha256sum -c fedora-coreos-40.20240616.3.0-vmware.x86_64.ova-CHECKSUM
```

### govc downloaden

```
# wget
wget https://github.com/vmware/govmomi/releases/latest/download/govc_$(uname -s)_$(uname -m).tar.gz
tar -zxvf govc_Linux_x86_64.tar.gz

# Einzeiler mit direkter Installation nach /usr/local/bin (root required)
curl -L -o - "https://github.com/vmware/govmomi/releases/latest/download/govc_$(uname -s)_$(uname -m).tar.gz" | tar -C /usr/local/bin -xvzf - govc
```

FCOS_OVA='fedora-coreos-40.20240616.3.0-vmware.x86_64.ova'
VM_NAME='<vmname>'
IFACE='ens192'
IPCFG="ip=<ip>::<gw>:<netmask>:${VM_NAME}:${IFACE}:off"
export GOVC_INSECURE=1
export GOVC_URL='https://<usr>:<passwort>@<vcenterurl>>
govc session.login -u '<user>:<passwort>@<vcenterurl>'

./govc find / -type ResourcePool
RESOURCE_POOL="<your resource pool>"
DATASTORE="<your data store"

find network
./govc ls -l

./govc ls -l <networkpath>

export GOVC_NETWORK="<your network>"



weiter untersuchen
./govc import.spec fedora-coreos-40.20240616.3.0-vmware.x86_64.ova




Im vCenter "OVA Vorlage bereitstellen" wählen
  -> Ignition config data encoding = gzip+base64
  -> Ignition config data = 



# Fedora 101

dnf repolist      - list repos
dnf updateinfo    - update infos
dnf update        - update
dnf search <...>  - search
dnf install gcc   - install gcc
dnf install clang - install clang
dnf repoquery -l <...>   - show files in package

## Custom Kernel

https://fedoraproject.org/wiki/Building_a_custom_kernel

