# Fedora

https://docs.fedoraproject.org/en-US/fedora-server/virtualization/vm-install-cloudimg-centos9/

Download Image

qemu-img  info CentOS-Stream-GenericCloud-9-20220315.0.x86_64.qcow2


# Fedora CoreOS

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

