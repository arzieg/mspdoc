# Debian Allgemein

## APT

https://blog.packagecloud.io/apt-cheat-sheet/

Definierte Repositories:
    /etc/apt/source.list
    apt policy

Update:
    sudo apt-get update && sudo apt-get dist-upgrade

apt-get
    * install and --reinstall
    * remove
    * purge or --purge
    * upgrade
    * update
    * clean and autoclean

apt-get install --only-upgrade git  - einzelnes Paket aktualisieren

apt-cache
    * pkgnames
    * search show

aptitude search <package>


dpkg
    * --list or -l 
    * -L                    list files in package
    * --install
    * --remove
    * --purge
    * --update
    * --contents, -c <debfile>

dpkg-query -L <package>   list files in package
dpkg -s <package>         list metainformation of package
apt show <package>        list metainformation of package


## Problem, wenn im Prozess des Zertifikat aufgebrochen wird. 

Machbar wäre, die Prüfung zu ignorieren. 
https://askubuntu.com/questions/1095266/apt-get-update-failed-because-certificate-verification-failed-because-handshake

```
apt-get -o Acquire::https::Verify-Peer=false update
apt-get -o Acquire::https::Verify-Peer=false install kubectl
```


## Problem in WSL beim Update

Process: sudo apt upgrade

Fehler: `Failed to take /etc/passwd lock: Invalid argument`

Workarround: 

```
    sudo mv /var/lib/dpkg/info/systemd.postinst /tmp/
    sudo apt install systemd=255.4-1ubuntu8.10 systemd-sysv=255.4-1ubuntu8.10
    sudo apt update && sudo apt upgrade
    sudo mv /tmp/systemd.postinst /var/lib/dpkg/info/
```
