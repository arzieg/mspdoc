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

https://github.com/iovisor/bcc/blob/master/INSTALL.md

### Linux Mint

```
Das nicht -> echo deb http://cloudfront.debian.net/debian sid main >> /etc/apt/sources.list
sudo apt-get install -y bpfcc-tools libbpfcc libbpfcc-dev linux-headers-$(uname -r)
```

PYTHONPATH anpassen in .bashrc
```
export PYTHONPATH=$(dirname `find /usr/lib -name bcc`):$PYTHONPATH
``` 

### Fedora

`sudo dnf install bcc`


# eBPF

## BPF Maps

A map is a data structure that can be accessed from an eBPF program and from user
space.

Maps can be used to share data among multiple eBPF programs or to communicate between a user space application and eBPF code running in the kernel. Typical uses include the following:
* User space writing configuration information to be retrieved by an eBPF program
* An eBPF program storing state, for later retrieval by another eBPF program (or a future run of the same program)
* An eBPF program writing results or metrics into a map, for retrieval by the user space app that will present results
There are various types of BPF maps defined in Linux’s uapi/linux/bpf.h file, and there is some information about them in the kernel docs. In general they are all key–value stores.

There are map types that are optimized for particular types of operations, such as first-in-first-out queues, first-in-last-out stacks, least-recently-used data storage, longest-prefix matching, and Bloom filters. Spin lock support for (some) maps (more CPU szenario) was added in kernel version 5.1.

### Hash Table Map

Die entsprechende Map wird über ein c-makro initialisiert: 

`BPF_HASH(counter_table);` Das Makro ist vmtl. in libbpf definiert (https://github.com/iovisor/bcc/blob/master/docs/reference_guide.md)




### Perf and Ring Buffer Maps

There is a newer construct called “BPF ring buffers” that are now generally preferred over BPF perf buffers, if you have a kernel of version 5.8 or above. Andrii Nakryiko discusses the difference in
his BPF ring buffer blog post (https://nakryiko.com/posts/bpf-ringbuf/).

You can think of a ring buffer as a piece of memory logically organized in a ring, with separate “write” and “read” pointers. Data of some arbitrary length gets written to wherever the write pointer is, with the length information included in a header for that data. The write pointer moves to after the end of that data, ready for the next write operation.

If the read pointer catches up with the write pointer, it simply means there’s no data to read.

If a write operation would make the write pointer overtake the read pointer, the data doesn’t get written and a drop counter gets incremented. Read operations include the drop counter to indicate whether data has been lost since the last successful read.

Macro: BPF_PERF_OUTPUT(output);
