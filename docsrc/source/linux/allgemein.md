.. _lnx_allg:


# Linux Allgemein


## Linux Kernel Release Nummerierung:

Quelle: https://www.youtube.com/watch?v=VfEHZnwdpw4&list=PL03Lrmd9CiGdBvVUXpZCKK88-Vpd5VwEo&index=5

  * ist keine semantische Nummerierung https://semver.org
  * X.YY.ZZ
  
    * X.YY = Major Version (not major/minor)
    * ZZ = Bugfixes
  * Alle 9-10 Wochen wird ein Major/Minor erstellt
  * 4.19.x -> 4.20.x upgrade can contain more features/breaking changes than 4.20.x->5.0.x
  * Repo vom Kernel: https://github.com/torvalds/linux
  * Typischerweise ist ein stable release 2-3 Monate aktiv, danach ist es EOL
  * Daher gibt es longterm stable releases
  
    * Backport ca. 2 Jahre
    * Release jedes Jahr
  
## Semantische Versionierung

Quelle: https://semver.org/

Given a version number MAJOR.MINOR.PATCH, increment the:

1. MAJOR version when you make incompatible API changes
2. MINOR version when you add functionality in a backward compatible manner
3. PATCH version when you make backward compatible bug fixes

Additional labels for pre-release and build metadata are available as extensions to the MAJOR.MINOR.PATCH format.

Is there a suggested regular expression (RegEx) to check a SemVer string?
There are two. One with named groups for those systems that support them (PCRE [Perl Compatible Regular Expressions, i.e. Perl, PHP and R], Python and Go).

See: https://regex101.com/r/Ly7O1x/3/

And one with numbered capture groups instead (so cg1 = major, cg2 = minor, cg3 = patch, cg4 = prerelease and cg5 = buildmetadata) that is compatible with ECMA Script (JavaScript), PCRE (Perl Compatible Regular Expressions, i.e. Perl, PHP and R), Python and Go.

See: https://regex101.com/r/vkijKf/1/


## Kleine Helferlein:

du -hs * | sort -hr                                         Größe der Verzeichnisse
find /tmp -type f -size +100000k -exec ls -lh {} \;         suche Dateien größer 10 MB  
find /oracle/base/ -name "log_*.xml" -mtime +30 -exec ls -l {} \;
find . -type f -size +10000k -name *.log -exec ls -lh {} \;
find /opt/mapr/hadoop/hadoop-2.7.0/logs/userlogs -mtime +30 -exec rm -rf {} \;
find . -mtime +7 -exec rm -rf {} \;
find . -mtime +30 -exec ls -lh {} \;
find  / -type f -mtime +30 -exec ls -l {} \;
find /tmp -type f -size +50000k| xargs ls -lahS
find <verzeichnis> -type d -links 2 -> zeige Verzeichnisse an, die keine Unterverzeichnisse haben
tar -zcvpf backup_`date +"%Y%m%d_%H%M%S"`.tar.gz `find /opt/mapr/hadoop/hadoop-2.7.0/logs/userlogs -mtime +120`    -> backup, alles was älter als 120 Tage ist
tar -zcvpf hostname_log.tgz /opt/mapr/logs
 
tar -cf - /quelle | ssh user@example tar -xvf - -C /ziel/     - Das Verfahren hat Probleme mit symlinks, die ggfs. noch nicht angelegt sind
mit bzip-Komprimmierung: 				tar -cjf - /quelle | ssh user@example tar -xjvf - -C /ziel/
beibehaltung der Dateiberechtigungen: 	tar pcf - /quelle | ssh user@example tar -pxvf - -C /ziel/
mit bzip-Komprimmierung+Dateiberech.	tar -pcjf - /quelle | ssh user@example tar -pxjvf - -C /ziel/
tar cpf - /some/important/data | ssh user@destination-machine "tar xpf - -C /some/directory/"  -stabil, läuft gut
  
find /mnt/dump -type f -exec md5sum {} \;> /tmp/checksums.md 
md5sum -c /tmp/checksums.md
 

## RDP

zypper install -t pattern patterns-xfce-xfce_basis patterns-xfce-xfce patterns-xfce-xfce_office
zypper in xrdp
vi /etc/sysconfig/windowsmanager
  Change DEFAULT_WM to DEFAULT_WM="xfce"


## User


useradd -m -d /sapmnt/S77/s77adm -s /usr/bin/csh -c "SAP System Administrator" -u 56069 -g 1001 -G dba,oper,sapinst,asmoper,asmdba,oinstall s77adm


## ipmi
`ipmitool power status -I lanplus -H timo-ipmi -U hacluster -P "xxx" -L Operator`