# RAC on Podman

https://docs.oracle.com/en/database/oracle/oracle-database/19/racpd/host-preparation-oracle-rac-podman.html

https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-AboutPodmanBuildahandSkopeo.html#podman-about

## Requirements

* Podman > 4.0.2
* Oracle Linux 8 Container image (oraclelinux:8)

## VM
https://docs.oracle.com/en/database/oracle/oracle-database/19/racpd/target-configuration-oracle-rac-podman.htm
Ziel: zwei Container auf einem Podman-Host

32 GB RAM

Disklayout
80 GB OS
SWAP 16 GB
scratch1 80 GB
scratch2 80 GB
containers 100 GB xfs
sdd-g je 50 GB



## Install

```
zypper in podman
zypper in podman-docker
zypper in buildah
zypper in skopeo
```

