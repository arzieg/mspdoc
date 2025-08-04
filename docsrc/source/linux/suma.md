# SUSE Manager


## Spacewalk
start: `spacewalk-service start`
status: `spacewalk-service status`
stop: `spacewalk-service stop`


## Zypper


zypper lp --cve      Anzeigen der Patche inkl. cve Nummer
zypper lp --cve -a   Anzeige allter Patche und welche nicht implementiert werden müssen, da bereits im Update der SUSE enthalten.

## bootstap-repo

Für das bootstrapen muss ein Repo auf dem SUSE Manager angelegt werden

mgr-create-bootstrap-repo --list --force   (--force zeige auch die unsupporteten Versionen an)
mgr-create-bootstrap-repo --create SLE-15-SP3-x86_64 --force  (--force mach es )


## safe.directory

Bei der git integration kann ein Fehler auftreten, dass vom Repo nicht gelesen werden kann, da der ownership nicht richtig wäre. 
Als Lösung soll man `git config --global --add safe.directory <repoort>` eingeben. 

Hier ist zu beachten, dass in der ~/.gitconfig der Eintrag zum safe-directory angelegt wird

`git config --global --add safe.directory '*'`

Ich hatte aber auch schon das Phänomen, dass das nicht geholfen hat. Dann musste man die ~/.gitconfig nach /etc/gitconfig kopieren (als Vorlage für alle User)


## register host
curl -Sks https://<url>/pub/bootstrap/salt-bootstrap-release-50-sle155-latest.sh | /bin/bash


## regeneration yum cache

spacecmd -> softwarechannel_regenerateyumcache channels
Überwachung mit: taskotop oder tail -f /var/log/rhn/rhn_taskomatic_daemon.log
------------------
# SUMA 5

mgrctl term  - connect to pod


## SUSE Mgr. Proxy

mgrpxy start uyuni-proxy-pod

