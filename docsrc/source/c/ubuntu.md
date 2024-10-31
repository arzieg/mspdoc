# Ubuntu Cloud Image in KVM

https://gist.github.com/mikejoh/c0c9be0275a1b4867c15994b52a98237

0. Install cloud-image-utils

```
apt install cloud-image-utils
```

1. Export 

```
export MAC_ADDR=$(printf '52:54:00:%02x:%02x:%02x' $((RANDOM%256)) $((RANDOM%256)) $((RANDOM%256)))
export INTERFACE=eth001
export IP_ADDR=192.168.122.101
export VM_NAME=ubuntu01
export UBUNTU_RELEASE=focal
export VM_IMAGE=$UBUNTU_RELEASE-server-cloudimg-amd64-disk-kvm.img
```

2. Download 

`wget https://cloud-images.ubuntu.com/$UBUNTU_RELEASE/current/$UBUNTU_RELEASE-server-cloudimg-amd64-disk-kvm.img`

3. create Diskimage

```
qemu-img create -F qcow2 -b `pwd`/$VM_IMAGE -f qcow2 /var/lib/libvirt/images/$VM_NAME.qcow2 10G
```

4. Create cloud-init files

```
cat >network-config <<EOF                                                             
ethernets:    
    $INTERFACE:
        addresses:
        - $IP_ADDR/24
        dhcp4: false
        gateway4: 192.168.122.1
        match:
            macaddress: $MAC_ADDR
        nameservers:
            addresses:
            - 1.1.1.1
            - 8.8.8.8
        set-name: $INTERFACE
version: 2
EOF

cat >user-data <<EOF
#cloud-config
hostname: $VM_NAME
manage_etc_hosts: true
users:
  - name: ubuntu
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: users, admin
    home: /home/ubuntu
    shell: /bin/bash
    lock_passwd: false
ssh_pwauth: true
disable_root: false
chpasswd:
  list: |
    ubuntu:ubuntu
  expire: False

ssh_authorized_keys:
  ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDLZ6Uk0Ijq2l0t7jiR2wJUxf3mj67/8frQjqdEYBcbKBNDoRkfbr0seX/aboYGPXsRtkTnpVZx/v2WBKWrAzGyehIH25wUTxBXj+pl/D1soGEm4hnioApN5xO3DiWLVZNm3nF5Of3Gu0dJdFE1AWb3hmBoIitbcX2lNUCb3qukBbhhBs2hZiokGQRvtfUAHwdLiwGNHyqLN6U/VEjze4/FPcnkaA/xucLMGxxL5HWUewVzgvx/fKUGYPqJ87SLk3D4sbcJrncXcNOQo+QCIMXmkwQk/tKj06n0pQ+SUBkTVIJBshrP4U/G98wiIs4n4vk+V35FJrPjE6UKLH5XMXDwQNZ8Hl0Xk5TW3GLmc8LeqIycihMSD0Z05gXRoF/0SKq5pAj8r4x7JO2E9fFeqaUsNyU5aw/rCFVngbm5lbJ3/qkSMME/KIqgTthJ5E0t171LHZUIVN20l4wJ9uja3xtghnA68AhpKPiLhrPz8YXJj2B4D2pFMuO05kEGy/RGQlV4rfUCkaLz7QQ7NKGLT5ccb2v230oPrRWAIPM2RLXUMmX7Y8tB7BVNQmUholdhwFyRLwSFsK1Sf0SPADVegzzPP6AhWq5k6vv095zd9xgbAeIWPEytoV2FLDH+GWchE0nkp3zLXaDLkFck/ON407imxGXvNnN7yrHQ5KntWAzgcQ== arne.ziegler@gmx.net
EOF
touch meta-data
```

5. Attach cloud-init to the image

```
cloud-localds -v --network-config=network-config `pwd`/$VM_NAME-seed.qcow2 user-data meta-data
```

6. create vm

```
virt-install --connect qemu:///system \
  --virt-type kvm \
  --name $VM_NAME \
  --vcpus 2 \
  --os-type linux \
  --os-variant ubuntu20.04 \
  --disk path=/var/lib/libvirt/images/$VM_NAME.qcow2,device=disk \
  --disk path=`pwd`/$VM_NAME-seed.qcow2,device=disk \
  --import \
  --network network=default,model=virtio,mac=$MAC_ADDR \
  --noautoconsole
  ```

Memory ( --memory 1024  ) hat er nicht genommen. Daher default = 4096. Danach dann manuell runtersetzen auf 1024.

7. SSH to the VM or attach the console, use admin as username and password:

```
virsh console $VM_NAME
ssh ubuntu@${IP_ADDR}
```

8. Shutdown

```
virsh shutdown $VM_NAME
```

9. Destroy the VM

```
virsh destroy $VM_NAME
virsh undefine $VM_NAME
```
