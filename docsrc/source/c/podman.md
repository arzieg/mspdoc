# Podman

https://podman.io/

bootc container images

## Install
```
sudo apt-get install podman
podman version
vi /etc/containers/registries.conf
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


### /usr/share/containers/mounts.conf

```
/usr/share/rhel/secrets:/run/secrets
```
In diesem Beispiel wird der Inhalt von /usr/share/rhel/secrets in den container nach /run/secrets gemountet (eher kopiert, da kein Mount)


### /usr/share/containers/seccomp.json
seccomp.json contains the whitelist of seccomp rules to be allowed inside of containers. This file is usually provided by the containers-common package.

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


