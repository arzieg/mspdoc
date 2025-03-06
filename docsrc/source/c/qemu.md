# qemu

https://u-labs.de/portal/qemu-kvm-libvirt-fuer-einsteiger-so-richtest-du-deine-erste-virtuelle-maschine-ein-mit-grafischer-oberflaeche/

```
# Installiert alle Abhaengigkeiten - fuer Einsteiger/Workstations
sudo apt install qemu-system libvirt-daemon-system qemu-utils virt-manager

# Verhindert die Installation grafischer Pakete - Auf Servern zu empfehlen (ohne virt-manager GUI)
sudo apt install --no-install-recommends qemu-system libvirt-clients libvirt-daemon-system qemu-utils dnsmasq-base

# Ausfuehrliche Pruefung des Systems auf Kompatibilitaet nach Paketinstallation
sudo virt-host-validate
```

User zu Gruppe hinzufügen
```
sudo usermod -a -G libvirt $USER
sudo usermod -a -G kvm $USER
```

Services
```
sudo systemctl status libvirtd
```

## Qemu on SLES

zypper in qemu qemu-kvm
modprobe kvm
lsmod |grep kvm
mkdir qemuimages
cd qemuimages
wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
python3 -m http.server

qemu-system-x86_64                                            \
    -net nic                                                    \
    -net user                                                   \
    -machine accel=kvm:tcg                                      \
    -m 1024                                                     \
    -nographic                                                  \
    -hda jammy-server-cloudimg-amd64.img                        \
    -smbios type=1,serial=ds='nocloud;s=http://10.0.2.2:8000/'