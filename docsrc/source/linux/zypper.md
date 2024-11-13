# Zypper 

zypper se --type pattern            - List avaiable pattern
zypper info --requires <pattern>    - welche Software ist in dem pattern
zypper in -t pattern <pattern>      - installiere ein pattern


## rpm Helfer

https://www.tecmint.com/20-practical-examples-of-rpm-commands-in-linux/

rpm --checksig hardinfo-2.0.11-FedoraLinux-39.x86_64.rpm        Check signature
rpm -ivh hardinfo-2.0.11-FedoraLinux-39.x86_64.rpm              Install packages
rpm -qpR hardinfo-2.0.11-FedoraLinux-39.x86_64.rpm              Check Dependencies of RPM Package Before Installing
rpm -ivh --nodeps hardinfo-2.0.11-FedoraLinux-39.x86_64.rpm     Install RPM Package Without Dependencies
rpm -ql hardinfo                                                Find Where RPM Files are Installed
rpm -qa --last                                                  List Recently Installed RPM Packages
rpm -Uvh hardinfo-2.0.11-FedoraLinux-39.x86_64.rpm              Upgrade a RPM Package
rpm -evv hardinfo                                               Remove a RPM Package
rpm -ev --nodeps hardinfo                                       Remove an RPM Package Without Dependencies
rpm -qf /usr/bin/htpasswd                                       Find RPM Package of a Specific File
rpm -qi hardinfo                                                Query Information of Installed RPM Package
rpm -qip sqlbuddy                                               Get the Information of RPM Package Before Installing
rpm -qdf /usr/bin/vmstat                                        Query Documentation of Installed RPM Package
rpm -Vp sqlbuddy-1.3.3-1.noarch.rpm                             Verify a RPM Package
rpm -Va                                                         Verify all RPM Packages
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-8              Import an RPM GPG Key
rpm -qa gpg-pubkey*                                             List all Imported RPM GPG Keys


## Rebuild Corrupted RPM Database

```
cd /var/lib
rm __db*
rpm --rebuilddb
rpmdb_verify Packages
```

## Remove old kernels

Wird definiert in /etc/zypp/zypp.conf

```
...
## Default: Do not delete any kernels if multiversion = provides:multiversion(kernel) is set
multiversion.kernels = latest,running
...
```

Nach einem zypper up wird eine Datei /boot/do_purge_kernels angelegt. Nach einem reboot erfolgt One-Shot: purge-kernels.service
Den kann man auch manuell anstossen: `systemctl restart purge-kernels.service`

In Azure ist der nicht installiert. Da könnte man manuell ein `sudo zypper purge-kernels` absetzen oder das Paket `zypper in purge-kernels-service` installieren.
