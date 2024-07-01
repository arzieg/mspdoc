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
