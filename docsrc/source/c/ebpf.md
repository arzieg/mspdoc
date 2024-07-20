.. _ebpf_allg:

# EBPF

EBPF Einstieg: https://ebpf.io/

## Demo libbpf

blog: https://nakryiko.com/posts/libbpf-bootstrap/
github: libbpf-bootstrap, demo BPF applications: https://github.com/libbpf/libbpf-bootstrap

github: XDP Tutorial https://github.com/xdp-project/xdp-tutorial
github: Utilities and example programs for use with XDP, https://github.com/xdp-project/xdp-tools

API:
https://libbpf.readthedocs.io/en/latest/api.html


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

`sudo dnf in libbpf-devel`   // für /usr/include/bpf/bpf_helper_defs.h

`sudo dnf install clang`     // besser als gcc für ebpf? 

`sudo dnf install bpftool`

Bei c-Programmen ist darauf zu achten, dass \#include \<linux/types.h\> als erste Zeile enthalten ist.  


# eBPF

## Allgemein

Linux Tracing Events: /sys/kernel/debug/tracing/events/



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

### Function Calls

in the early days, eBPF programs were not permitted to call functions other than helper functions. To work around this, programmers have directed the compiler to “always inline” their functions, like this:

`static __always_inline void my_function(void *ctx, int val)`

Starting from Linux kernel 4.16 and LLVM 6.0, the restriction requiring functions to be inlined was lifted so that eBPF programmers could write function calls more naturally. However, this feature, called “BPF to BPF function calls” or “BPF subprograms,” isn’t currently supported by the BCC framework.

Eine andere Form sind **tail calls**

Tail calls are by no means exclusive to eBPF programming. The general motivation behind tail calls is to avoid adding frames to the stack over and over again as a function is called recursively, which
can eventually lead to stack overflow errors. If you can arrange your code to call a recursive function as the last thing it does, the stack frame associated with the calling function isn’t really doing anything useful. Tail calls allow for calling a series of functions without growing the stack. This is particularly useful in eBPF where the stack is limited to **512 bytes**.

Signature: `long bpf_tail_call(void *ctx, struct bpf_map *prog_array_map, u32 index)`

* ctx allows passing the context from the calling eBPF program to the callee.
* prog_array_map is an eBPF map of type BPF_MAP_TYPE_PROG_ARRAY, which holds a set of file descriptors that identify eBPF programs.
* index indicates which of that set of eBPF programs should be invoked.

This helper is somewhat unusual in that if it succeeds, it never returns. The currently running eBPF program is replaced on the stack by the program being called.

Im Programm müssen dann die einzelnen c-Programme (=Funktionen hier) aufrufbar sein. Im python code werden diese einzelnen ebpf-programme dann registriert

```
...
b = BPF(text=program)                                              
b.attach_raw_tracepoint(tp="sys_enter", fn_name="hello")           <- Eintrittspunkt

ignore_fn = b.load_func("ignore_opcode", BPF.RAW_TRACEPOINT)       <- einzelne Programme registrieren, müssen alle vom selben "Typ" sein
exec_fn = b.load_func("hello_exec", BPF.RAW_TRACEPOINT)
timer_fn = b.load_func("hello_timer", BPF.RAW_TRACEPOINT)
...
b = BPF(text=program)                                              
b.attach_raw_tracepoint(tp="sys_enter", fn_name="hello")           

ignore_fn = b.load_func("ignore_opcode", BPF.RAW_TRACEPOINT)       
exec_fn = b.load_func("hello_exec", BPF.RAW_TRACEPOINT)
timer_fn = b.load_func("hello_timer", BPF.RAW_TRACEPOINT)
```



## Network - Hello World

### code-Beispiel

```
#include <linux/types.h>
#include <bpf/bpf_helpers.h>
#include <linux/bpf.h>

int counter = 0;

SEC("xdp")

int hello(void *ctx) {
  bpf_printk("Hello World %d", counter);
  counter++;
  return XDP_PASS;
}

char LICENSE[] SEC("license") = "Dual BSD/GPL";
```

*Makefile*

```
hello.bpf.o: %o: %c
	clang \
		-target bpf \
		-I/usr/include/linux/ \
		-I/usr/include/bpf/ \
        -I/usr/src/kernels/${shell uname -r}/tools/lib/ \
		-g \
		-O2 -c $< -o $@
```

*Fileinfo*

```
fedora@localhost c3]$ file hello.bpf.o
hello.bpf.o: ELF 64-bit LSB relocatable, eBPF, version 1 (SYSV), with debug_info, not stripped
```

*Objectdump*

```
[fedora@localhost c3]$ llvm-objdump -S hello.bpf.o

hello.bpf.o:	file format elf64-bpf

Disassembly of section xdp:

0000000000000000 <hello>:
;   bpf_printk("Hello World %d", counter);
       0:	18 06 00 00 00 00 00 00 00 00 00 00 00 00 00 00	r6 = 0x0 ll
       2:	61 63 00 00 00 00 00 00	r3 = *(u32 *)(r6 + 0x0)
       3:	18 01 00 00 00 00 00 00 00 00 00 00 00 00 00 00	r1 = 0x0 ll
       5:	b7 02 00 00 0f 00 00 00	r2 = 0xf
       6:	85 00 00 00 06 00 00 00	call 0x6
;   counter++;
       7:	61 61 00 00 00 00 00 00	r1 = *(u32 *)(r6 + 0x0)
       8:	07 01 00 00 01 00 00 00	r1 += 0x1
       9:	63 16 00 00 00 00 00 00	*(u32 *)(r6 + 0x0) = r1
;   return XDP_PASS;
      10:	b7 00 00 00 02 00 00 00	r0 = 0x2
      11:	95 00 00 00 00 00 00 00	exit
```
eBPF instructions are generally 8 bytes long, and since on a 64-bit platform each memory location can hold 8 bytes.
Unofficial ebf spec: https://github.com/iovisor/bpf-docs/blob/master/eBPF.md

BPF Standard Documentation: https://github.com/ietf-wg-bpf/ebpf-docs?tab=readme-ov-file

### Objectfile laden

`sudo bpftool prog load hello.bpf.o /sys/fs/bpf/hello`

`sudo ls /sys/fs/bpf`

### Details 

```
sudo bpftool prog list

61: xdp  name hello  tag d35b94b4c0c10efb  gpl
	loaded_at 2024-07-10T19:10:03+0000  uid 0
	xlated 96B  jited 72B  memlock 4096B  map_ids 16,17
	btf_id 66
```

```
sudo bpftool prog show id 61 --pretty
{
    "id": 61,
    "type": "xdp",
    "name": "hello",
    "tag": "d35b94b4c0c10efb",
    "gpl_compatible": true,
    "loaded_at": 1720638603,
    "uid": 0,
    "orphaned": false,
    "bytes_xlated": 96,
    "jited": true,
    "bytes_jited": 72,
    "bytes_memlock": 4096,
    "map_ids": [16,17
    ],
    "btf_id": 66
}
```

Das bpf Programm tag ist einmalig im System. 

*Anzeige des übersetzten BPF Programms:* 

```
fedora@localhost c3]$ sudo bpftool prog dump xlated name hello
int hello(void * ctx):
; bpf_printk("Hello World %d", counter);
   0: (18) r6 = map[id:16][0]+0
   2: (61) r3 = *(u32 *)(r6 +0)
   3: (18) r1 = map[id:17][0]+0
   5: (b7) r2 = 15
   6: (85) call bpf_trace_printk#-108704
; counter++;
   7: (61) r1 = *(u32 *)(r6 +0)
   8: (07) r1 += 1
   9: (63) *(u32 *)(r6 +0) = r1
; return XDP_PASS;
  10: (b7) r0 = 2
  11: (95) exit
```

*Anzeige des Just in Time Interpretercodes:*

```
[fedora@localhost c3]$ sudo bpftool prog dump jited name hello
int hello(void * ctx):
bpf_prog_d35b94b4c0c10efb_hello:
; bpf_printk("Hello World %d", counter);
   0:	endbr64
   4:	nopl   0x0(%rax,%rax,1)
   9:	xchg   %ax,%ax
   b:	push   %rbp
   c:	mov    %rsp,%rbp
   f:	endbr64
  13:	push   %rbx
  14:	movabs $0xffffb2b24036a000,%rbx
  1e:	mov    0x0(%rbx),%edx
  21:	movabs $0xffff9e7d47f03cf8,%rdi
  2b:	mov    $0xf,%esi
  30:	call   0xfffffffff71174a8
; counter++;
  35:	mov    0x0(%rbx),%edi
  38:	add    $0x1,%rdi
  3c:	mov    %edi,0x0(%rbx)
; return XDP_PASS;
  3f:	mov    $0x2,%eax
  44:	pop    %rbx
  45:	leave
  46:	ret
  47:	int3
```
### Attach to a event

```sudo bpftool net attach xdp id 61 dev enp1s0

sudo bpftool net list

xdp:
eth0(2) driver id 61

tc:

flow_dissector:

netfilter:
```

ip link zeigt das bpf Programm was am Interface eth0 gebunden ist

```
[fedora@localhost c3]$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 xdp qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:18:a1:63 brd ff:ff:ff:ff:ff:ff
    prog/xdp id 61 
    altname enp1s0
```

Zeige die Trace-Informationen an (je empfangenen IP Paket ein Traceeintrag)

`sudo cat /sys/kernel/debug/tracing/trace_pipe` oder `sudo bpftool prog tracelog`

### Zugriff auf Daten vom eBPF Programm über bpftool

eBPF Maps sind Datenstrukturen, die vom User Space angesprochen werden können. Diese Maps können z.B. verwendet werden, um Stati auszulesen. 

Über `sudo bpftool prog list` listet man die eBPF Programme. In der Kurzzusammenfassung werden die mapid's ausgegeben. 

Über 

```
[fedora@localhost c3]$ sudo bpftool map list
16: array  name hello.bss  flags 0x400
	key 4B  value 4B  max_entries 1  memlock 8192B
	btf_id 66
17: array  name hello.rodata  flags 0x80
	key 4B  value 15B  max_entries 1  memlock 264B
	btf_id 66  frozen
```

wird die map list ausgegeben, u.a. auch die beiden Maps für das Beispielprogramm. Die Werte können ausgelesen werden (aber nur, wenn mit der
DEBUG Option (-g) das BPF Programm compliert wurde): 

```
[fedora@localhost c3]$ sudo bpftool map dump name hello.bss  oder sudo bpftool map dump id 16
[{
        "value": {
            ".bss": [{
                    "counter": 34196
                }
            ]
        }
    }
]
```

### Detach eBPF Programm

Das Programm bleibt weiterhin im Kernel geladen! Es wird nur vom Netzwerkinterface getrennt. 

```
sudo bpftool net detach xdp dev enp1s0

[fedora@localhost c3]$ sudo bpftool net list
xdp:

tc:

flow_dissector:

netfilter:
```

### Unload eBPF Programm

`rm /sys/fs/bpf/hello`

`sudo bpftool prog show name hello`



# CO-RE, BTF, and Libbpf

## Overview

CO-RE = Compile once, run everywhere

bpttool btf list           Liste der geladenen BTF Objekte
bpftool btf dump id <id>   Untersuchung der Struktur eines bpf Programms 
bpftool btf dump file /sys/kernel/btf/vmlinux format c > vmlinux.h                erzeuge eine header Datei aus der Struktur von vmlinux
bpftool gen skeleton hello-buffer-config.bpf.o > hello-buffer-config.skel.h       erzeuge eine header Datei aus der Objekt-Datei









