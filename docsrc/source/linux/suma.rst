.. _suma_allg:

################
SUSE Manager
################


Zypper
======

zypper lp --cve      Anzeigen der Patche inkl. cve Nummer
zypper lp --cve -a   Anzeige allter Patche und welche nicht implementiert werden müssen, da bereits im Update der SUSE enthalten.

bootstap-repo
==============

Für das bootstrapen muss ein Repo auf dem SUSE Manager angelegt werden

mgr-create-bootstrap-repo --list --force   (--force zeige auch die unsupporteten Versionen an)
mgr-create-bootstrap-repo --create SLE-15-SP3-x86_64 --force  (--force mach es )
