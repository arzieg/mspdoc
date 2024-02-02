.. _lvm_allg:

################
LVM
################

LVM
====
# Infos
    pvs
    vgs
    lvs
    lsscsi
    lsblk

# Filesystem erweitern (https://wiki.ez.edeka.net/display/DFVLOSWEB/Linux+Filesystemerweiterung)
# SuSE Note: https://www.suse.com/support/kb/doc/?id=000017762
echo "- - -" > /sys/class/scsi_host/host0/scan  (rescan-scsi-bus.sh -a bei vmware hilft häufig)
pvcreate /dev/sdd
vgextend vg_test /dev/sdd
lvresize –L +10G /dev/mapper/vg_test_lv_test  (ging beim letzten Mal nicht, SLES15, lvextend verwendet)
(lvextend -l +100%FREE /dev/mapper/vg_test_lv_test) <- nimm den freien Rest und wachse bis dahin
(lvextend -L +100G /dev/vg_spacewalk/lv_spacewalk) <- volume soll um 100 GB wachsen
(lvextend -L 100G /dev/vg_spacewalk/lv_spacewalk)  <- volume soll final 100 GB groß sein
(lvextend -r -l 38912 /dev/mapper/vg_system-lv_tmp) <- volume soll exact soviel extends haben
## blkid -> um was für ein Filesystem handelt es sich? 
## EXT FS
    resize2fs /dev/mapper/vg_test_lv_test 
## XFS
    xfs_growfs
    Bsp.: xfs_growfs /var/spacewalk
## Oder Alternativ ( Dann kann auch resize2fs o. xfs_growfs verzichtet werden
    lvresize -r –L +10G /dev/mapper/vg_test_lv_test
    

Volume erzeugen (https://www.thomas-krenn.com/de/wiki/LVM_Grundkonfiguration)
echo "- - -" > /sys/class/scsi_host/host0/scan
pvcreate /dev/sdd
vgcreate vg_sri /dev/sdd
lvcreate -n lv_sapmnt -L25G vg_sri
lvcreate -n lv_usrsap -L15G vg_sri

# lvcreate -n lv_opt -l100%FREE vg_opt -> komplette vg ausnutzen
mkfs.xfs /dev/vg_sri/lv_sapmnt
useradd -m -d /home/oracle -s /bin/bash -c "Oracle Database Administrator" -u 500 oracle -g oinstall -G dba,dasi
SLES12:  usermod -a -G <group> <user>
	
Anzeige Disks zu Volumes:
--------------------------
pvdisplay -C --separator '  |  ' -o pv_name,vg_name

    VG vg_orahome is using an old PV header, modify the VG to update
    -----------------------------------------------------------------
    vgck vg_orahome    -> zeigt das Problem
    vgck --updatemetadata vg_orahome    -> Problem fixen
    vgck vg_orahome    -> Problem sollte nicht mehr angezeigt werden
