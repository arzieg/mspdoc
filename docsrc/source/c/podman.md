# Podman

https://podman.io/

bootc container images

## Install
```
MINT:
sudo apt-get install podman
podman version
vi /etc/containers/registries.conf
```

```
SUSE:
zypper in podman
zypper in podman-docker
```

```
Oracle Linux 8
https://oracle-base.com/articles/linux/podman-install-on-oracle-linux-ol8

für azure:
dnf install cifs-utils

```


## Configuration

### /etc/containers/registries.conf

bootc-images: https://docs.fedoraproject.org/en-US/bootc/base-images/

```

unqualified-search-registries = ["registry.fedoraproject.org", "registry.access.redhat.com", "docker.io", "quay.io"]

[[registry]]
location="localhost:5000"
insecure=true
```

### Storage

By default, images are stored in the /var/lib/containers directory when Podman is run by the root user. For standard users, images are typically stored in $HOME/.local/share/containers/storage/. These locations conform to Open Container Initiative (OCI) specifications. The separation of the local image repository for standard users ensures that containers and images maintain the correct permissions and that containers can run concurrently without affecting other users on the system.

System-wide storage configuration is handled by editing the file at /etc/containers/storage.conf.
You can override the system-wide storage configuration by creating a separate $HOME/.config/containers/storage.conf configuration file for any user.


### /usr/share/containers/mounts.conf

```
/usr/share/rhel/secrets:/run/secrets
```
In diesem Beispiel wird der Inhalt von /usr/share/rhel/secrets in den container nach /run/secrets gemountet (eher kopiert, da kein Mount)


### /usr/share/containers/seccomp.json
seccomp.json contains the whitelist of seccomp rules to be allowed inside of containers. This file is usually provided by the containers-common package.


### signed Images
You can configure Podman to only trust images from a remote registry if they're signed and the provided signature can be validated against a locally stored public key. 

1. For each registry where you require signature validation, create a YAML format configuration file in /etc/containers/registries.d/ and provide the value for the sigstore for that registry.

```
vi /etc/containers/registries.d/oracle.yaml

docker:
  container-registry.oracle.com:
    sigstore: https://container-trust.oci.oraclecloud.com/podman
```

2. Download and store the public GPG key that must be used to validate signatures for images from the registry

als root

```
mkdir -p /etc/pki/containers
wget -O /etc/pki/containers/GPG-KEY-oracle https://container-trust.oci.oraclecloud.com/podman/GPG-KEY-oracle
```

3. Edit the container policy configuration to add the location of the public GPG key that must be used to validate the signatures for images that are pulled from a particular registry. The policy configuration is in JSON format and is at /etc/containers/policy.json.

```
{
  "default": [
    {
      "type": "insecureAcceptAnything"
    }
  ],
  "transports":
    {
      "docker-daemon":
        {
          "": [{"type":"insecureAcceptAnything"}]
        },
      "docker":
        {
          "container-registry.oracle.com": [
            {
              "type": "signedBy",
              "keyType": "GPGKeys",
              "keyPath": "/etc/pki/containers/GPG-KEY-oracle"
            }
          ]
        }
    }
}

```

4. Testen
Hier muss man einmal den keyPath austauschen in der policy.json, damit man einen Fehler erzeugt, dass der Key nicht funktioniert. Bei richtiger Konfiguration unterscheidet sich das ansonsten nicht von dem normalen podman pull ohne Signatur

```
podman pull oraclelinux:8

Resolved "oraclelinux" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull container-registry.oracle.com/os/oraclelinux:8...
Error: copying system image from manifest list: Source image rejected: No public keys imported
```



## 101 Befehle

```
podman ps             List 
podman start <ctr>    start container
podman stop <ctr>     stop container
podman rm <ctr>       remove container
podman run -it <imageurl> run image
podman run -it registry.access.redhat.com/rhel7/rhel /bin/bash
podman inspect [repository or image ID]    inspect image
podman --help         help
podman search oraclelinux
podman pull registry.host/repository/imagename:tag
podman pull oraclelinux:8  (funktioniert, da in /etc/containers/registries.conf.d/000-shortnames.conf definiert)

attach        Attach to the shell of a running container
auto-update   Automatically update containers with automatic updating enabled
build         Build an image from a Containerfile
commit        Create an image based on an edited container
container     Manager existing containers
cp            Copy files and folders between the container and host file system
create  --name <name>       Create a container without starting it
diff          View changes on the container's file system
events        Show Podman event logs for the running container
exec          Run a process in a specified container that's already running
export        Export a container's file system and contents to a compressed archive
generate      Generate systemd unit files and pod YAML files
healthcheck   Run a healthcheck on an existing container
history       Review the history of a specified image
image         Manages existing images
images        Lists images present on the host system
import        Import a compressed archive to create a container file system
info          Show system information for Podman
init          Initialize one or more containers
inspect       Display the existing configuration values for a container or image
kill          Stop running containers with a predefined signal
load          Load an image from a container archive file
login         Log in to a container registry
logout        Log out of a container registry
logs          Review logs for a container
              podman logs oracleshell
manifest      Create and edit manifest lists and image indexes
mount         Mount a running container's root file system
network       Manage networks that are accessible to containers
pause         Suspend all the processes in one or more containers
play          Start a Pod
pod           Create and manage pods
              pod stop <podname> = stop all containers in pod
              pod start <podname> = start all containers in pod
              pod ps = check running containers in pod
port          List network port mappings for a container
ps            List containers
              ps -a  = all
              ps -ap = all with pod information
pull          Pull an image from a container registry
push          Push an image to a container registry
restart       Restart one or more containers
rm            Remove one or more containers
rmi           Remove one or more images from local storage on the host system
run           Run a single command in a new container
              podman run --name=oracleshell -it oraclelinux:7-slim /bin/bash   (run a shell)
save          Save an image to a compressed archive
search        Search a container registry for an image
start         Start one or more containers
              podman start -ai oracleshell
stats         Display a real time stream of container resource usage statistics
stop          Stop one or more containers
system        Manage Podman configuration settings
tag           Add another name to an image in local storage on the host system
top           Display the running processes for a container
unmount       Unmount a running container's root file system
unpause       Resume all processes for one or more containers
unshare       Run a command as a specified user
untag         Remove a secondary name from an image in local storage on the host system
version       Show version information for Podman
volume        Manage container storage volumes  
  prune       Löschen von nicht mehr genutzten Volumes
wait          Block processes on one or more containers until a specified condition is fulfilled
```




## Löschen
https://access.redhat.com/solutions/7041142

```
for i in `podman ps --all --storage | sed '1d' | awk '{print$1}'`; do podman rm $i ; done
podman rmi --force -a
```

if the podman rmi command fails with an unmounting error then execute the following commands --
```
$ podman rmi --force -a
  Error: 2 errors occurred:
          * unmounting
"/var/lib/awx/.local/share/containers/storage/overlay/bb3619748d3c2abe117dfce7edd16541c3a8605ac8dc3e7d1c96cf75270f47d6/merged": invalid argument

$ podman unshare mount -t tmpfs none /var/lib/awx/.local/share/containers/storage/overlay/bb3619748d3c2abe117dfce7edd16541c3a8605ac8dc3e7d1c96cf75270f47d6/merged

$ podman rmi --force -a

$ podman ps --all --storage
  CONTAINER ID  IMAGE                     COMMAND     CREATED       STATUS      PORTS       NAMES
```

## Build

```
mkdir oshttp
```

Create Dockerfile
```
FROM quay.io/fedora/fedora-bootc:40
# The default package drops content in /var/www, and on bootc systems
# we have /var as a machine-local mount by default. Because this content
# should be read-only (at runtime) and versioned with the container image,
# we move it to /usr/share/www instead.
RUN dnf -y install httpd && \
    systemctl enable httpd && \
    mv /var/www /usr/share/www && \
    echo 'd /var/log/httpd 0700 - - -' > /usr/lib/tmpfiles.d/httpd-log.conf && \
    sed -ie 's,/var/www,/usr/share/www,' /etc/httpd/conf/httpd.conf
# Further, we also disable the default index.html which includes the operating
# system information (bad idea from a fingerprinting perspective), and crucially
# we inject our own content as part of the container image build.
# This is a key point: In this model, the webserver content is lifecycled exactly
# with the container image build, and you can change it "day 2" by updating
# the image. The content is underneath the /usr readonly bind mount - it
# should not be mutated per machine.
RUN rm /usr/share/httpd/noindex -rf
COPY index.html /usr/share/www/html
EXPOSE 80
```

create index.html file

```
podman build -t oshttp  .

podman images
podman run localhost/oshttp
```




## Podman pods

A pod is a collection of containers that are grouped together into a single namespace so that they can share resources, such as local networking to communicate with each other and interact. A pod can be used to group a set of services that you need to deploy a complete application.

```
podman pod create --name oraclepod
podman pod list 
podman pod rm oraclepod
```

To attach containers to a pod, use the --pod flag when you run the container.
```
podman run --pod oraclepod -it --rm oraclelinux:8
podman ps -ap
```



