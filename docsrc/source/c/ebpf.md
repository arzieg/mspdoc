.. _ebpf_allg:

# EBPF

EBPF Einstieg: https://ebpf.io/

## Demo libbpf

blog: https://nakryiko.com/posts/libbpf-bootstrap/
github: libbpf-bootstrap, demo BPF applications: https://github.com/libbpf/libbpf-bootstrap

github: XDP Tutorial https://github.com/xdp-project/xdp-tutorial
github: Utilities and example programs for use with XDP, https://github.com/xdp-project/xdp-tools


Monitoring-Project:

Coroot: https://github.com/coroot/coroot

Coroot is an open-source APM & Observability tool, a DataDog and NewRelic alternative. Powered by eBPF for rapid insights into system performance.
Monitor, analyze, and optimize your infrastructure effortlessly for peak reliability at any scale.


## Installing

Linux Mint

```
Das nicht -> echo deb http://cloudfront.debian.net/debian sid main >> /etc/apt/sources.list
sudo apt-get install -y bpfcc-tools libbpfcc libbpfcc-dev linux-headers-$(uname -r)
```

PYTHONPATH anpassen in .bashrc
```
export PYTHONPATH=$(dirname `find /usr/lib -name bcc`):$PYTHONPATH
`` 
