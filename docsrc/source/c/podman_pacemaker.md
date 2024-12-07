# Podman Pacemaker

p pull registry.fedoraproject.org/fedora-iot

container laufen auf dem selben Host

```
podman network create --subnet 192.5.0.0/16 newnet
podman network create -d bridge --subnet=10.0.62.0/24 --gateway=10.0.62.1 pub1_nw
podman network create -d bridge --subnet=10.0.61.0/24 ring1_nw
podman network create -d bridge --subnet=10.0.63.0/24 ring2_nw
```

Fehler mit dem Netzwerk CLI . 
```
WARN[0000] Error validating CNI config file /home/arne/.config/cni/net.d/pub1_nw.conflist: [plugin firewall does not support config version "1.0.0"] 
WARN[0000] Error validating CNI config file /home/arne/.config/cni/net.d/ring1_nw.conflist: [plugin firewall does not support config version "1.0.0"] 
WARN[0000] Error validating CNI config file /home/arne/.config/cni/net.d/ring2_nw.conflist: [plugin firewall does not support config version "1.0.0"] 
```

Lösung sein: https://www.michaelmcculley.com/updating-cni-plugins-for-podman-a-step-by-step-guide/





```
# podman create -t -i \
  --hostname pcm1 \
  --shm-size 2G \
  --volume /dev/shm \
  --dns-search=example.com \
  --privileged=false  \
  --cpuset-cpus 0-3 \
  --memory 2G \
  --memory-swap 2G \
  --cap-add=SYS_NICE \
  --cap-add=SYS_RESOURCE \
  --cap-add=NET_ADMIN \
  --cap-add=NET_RAW \
  --cap-add=AUDIT_WRITE \
  --cap-add=AUDIT_CONTROL \
  --restart=always \
  --systemd=true \
  --name pcm1 \
registry.fedoraproject.org/fedora-iot:latest
```