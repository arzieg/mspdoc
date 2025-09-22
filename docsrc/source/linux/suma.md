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

## start/stop
mgradm restart
mgradm start
mgradm stop

## SUSE Mgr. Proxy

mgrpxy start
mgrpxy status
mgrpxy stop
mgrpxy start uyuni-proxy-pod


Five SUSE Manager Proxy containers should be present and should be part of the proxy-pod container pod:
* proxy-salt-broker
* proxy-httpd
* proxy-tftpd
* proxy-squid
* proxy-ssh


## Update

```
transaction-update up
mgradm upgrade podman
```

## Paketinstallation
https://documentation.suse.com/smart/systems-management/html/Micro-transactional-updates/index.html

```
sudo transactional-update pkg install package_1 package_2 ...    - Installation von Package 1,2 ...
sudo transactional-update --continue 13 pkg install package_2    - Installation von Package2 in Snapshot 13
sudo transactional-update pkg install -t pattern pattern_name    - Installation eines Patterns

sudo transactional-update pkg remove package_name                - Package remove
sudo transactional-update pkg update package_name                - Update

sudo transactional-update patch                                  - Checks for available patches and installs them. The default option for this command is --non-interactive.
sudo transactional-update dup                                    - Performs an upgrade of your system. The default option for this command is --non-interactive.
sudo transactional-update up                                     - Updates installed packages to newer versions. The default option for this command is --non-interactive.
sudo transactional-update migration                              - The command migrates your system to a selected target. Typically, it is used to upgrade your system if it has been registered via SUSE Customer Center.
```

### Performing snapshots cleanup

You can use transactional-update to clean unused file system snapshots and unreferenced /etc overlay directories.

transactional-update recognizes the following cleanup commands:

```
cleanup-snapshots   The command marks all unused snapshots for removal by Snapper.

cleanup-overlays    The command removes all unused overlay layers of /etc in the /var/lib/overlay directory.

cleanup             The command combines the cleanup-snapshots and cleanup-overlays commands.
```

### Performing system rollback

GRUB 2 enables booting from btrfs snapshots and thus allows you to use any older functional snapshot in case the new snapshot does not work correctly.

When booting a snapshot, the parts of the file system included in the snapshot are mounted read-only; all other file systems and parts that are excluded from snapshots are mounted read-write and can be modified.

#### Rollback from a running system

```
sudo snapper list
sudo transactional-update rollback snapshot_number

To set the last working snapshot as the default one, run rollback last.
``` 

#### Rollback to a working snapshot

1. Reboot your system and select Start bootloader from a read-only snapshot.
2. Choose a snapshot to boot. The snapshots are sorted according to the date of creation, with the latest one at the top.
3. Log in to your system and check whether everything works as expected. The data written to directories excluded from the snapshots will stay untouched.
4. If the snapshot you booted into is not suitable for the rollback, reboot your system and choose another one.
5. If the snapshot works as expected, you can perform the rollback by running the following command:
    ```
    sudo transactional-update rollback
    ```
6. And reboot afterwards.

