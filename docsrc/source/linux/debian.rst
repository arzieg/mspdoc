.. _debian_allg:

################
Debian Allgemein
################

APT:
=====

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
