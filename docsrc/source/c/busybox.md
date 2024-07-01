# Busybox

https://medium.com/@ThyCrow/compiling-the-linux-kernel-and-creating-a-bootable-iso-from-it-6afb8d23ba22

## Download Kernel

```
export KERNELVERSION="6.9.7"
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-${KERNELVERSION}.tar.xz
tar vxf linux-${KERNELVERSION}.tar.xz
cd linux-${KERNELVERSION}
```

## Download make utils

```
apt install build-essentials flex libncurses5-dev bc libelf-dev bison
```

## Configure

```
# This will create the default config file for compiling Linux
make defconfig

# Customize kernel
make menuconfig
```
Man kann auch aus less /boot/config-* die Datei kopieren als Vorlage

## compile
```
make -j $(nproc)
```
Dauer bei 13th Gen Intel(R) Core(TM) i5-1335U, ca. 5 Min.


## create minimal file system with busybox
Hierfür wird busybox verwendet. 

```
BBVERSION="1.36.1"
wget https://busybox.net/downloads/busybox-${BBVERSION}.tar.bz2
tar xvf busybox-${BBVERSION}.tar.bz2
cd busybox-${BBVERSION}
```

*Just like Linux kernel, run make defconfig to make a default configuration for the BusyBox. Then use make menuconfig to bring the TUI and edit the configs. The option which you must edit is located in Settings, and then Build static binary (no shared libs). The reason that you want to enable this option is that we don’t want to compile Glibc for our Linux distro.*

```
make defconfig
make menuconfig
 -> settings -> build static binary
```

make
```
make -j $(nproc)
```

check des Ergebnisses (man achte auf statically linked)
``` 
$ file busybox
busybox: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, BuildID[sha1]=482817b374b1cf45b47324bf6ba6f1e9d7917db6, for GNU/Linux 3.2.0, stripped
```

### excurse Cross Compiling For 32-Bit Systems

If you choose to compile the Linux kernel for 32-bit, you also want to compile the BusyBox for 32bit systems; To do so, you have to at first get the gcc i686 compiler using this command on Ubuntu:

```
apt install gcc-i686-linux-gnu
```

Now run the following commands to configure and compile the BusyBox:

```
make ARCH=i686 CROSS_COMPILE=i686-linux-gnu- defconfig # Generates the default config
make ARCH=i686 CROSS_COMPILE=i686-linux-gnu- menuconfig # Enable the static linking
make ARCH=i686 CROSS_COMPILE=i686-linux-gnu- -j $(nproc) # Compile BusyBox
```

Now lets run the file busybox again and see the results. Hier muss 32Bit auftauchen

## create File System

```
make install
```
Es wird ein Verzeichnis _install erzeugt

```
_install$ tree -d
.
├── bin
├── sbin
└── usr
    ├── bin
    └── sbin
```

Weitere Verzeichnisse erstellen
```
mkdir dev proc sys
```

Erstelle init File
```
vi init

#!/bin/sh
mount -t devtmpfs none /dev
mount -t proc none /proc
mount -t sysfs none /sys
echo "Welcome to my Linux!"
exec /bin/sh
```

```
chmod +x init
```

erzeuge das Filesystem
```
find . -print0 | cpio --null -ov --format=newc | gzip -9 > ../initramfs.cpio.gz
```
Es wird ein file initramfs.cpio.gz eine Verzeichnisebene höher erzeugt.

## Test Kernel mit qemu

```
pwd
# home/arne/dev/image/busybox-1.36.1

cp ../linux-6.9.7/arch/x86_64/boot/bzImage .

qemu-system-x86_64 -kernel bzImage -initrd initramfs.cpio.gz
```

Mit ls einmal prüfen, ob etwas angezeigt wird. 


## create ISO

Erzeuge ein Verzeichnis und kopiere Dateien: 

```
pwd
# /home/arne/dev/image/busybox-1.36.1
mkdir iso
cd iso
mkdir boot
mkdir boot/grub
cd boot
cp ../../bzImage .
cp ../../initramfs.cpio.gz .
```

Erzeuge grub config. Hier fallabhängig 

Directory efi existiert, dann wird efi benutzt, andernfalls bios.
```
ls -ld /sys/firmware/efi
```

Now we have to configure the grub itself. Create a file named grub.cfg in grub folder of the boot folder. If your host is booted using BIOS (and thus the output ISO is BIOS too) put these lines in the config file:

vi boot/grub/grub.cfg

```
set default=0
set timeout=10
menuentry 'myos' --class os {
    insmod gzio
    insmod part_msdos
    linux /boot/bzImage
    initrd /boot/initramfs.cpio.gz
}
```

If you are using UEFI, put these lines in it:

```
set default=0
set timeout=10
# Load EFI video drivers. This device is EFI so keep the
# video mode while booting the linux kernel.
insmod efi_gop
insmod font
if loadfont /boot/grub/fonts/unicode.pf2
then
        insmod gfxterm
        set gfxmode=auto
        set gfxpayload=keep
        terminal_output gfxterm
fi
menuentry 'myos' --class os {
    insmod gzio
    insmod part_msdos
    linux /boot/bzImage
    initrd /boot/initramfs.cpio.gz
}
```

erzeuge das ISO

```
grub-mkrescue -o busybox.iso iso/
```







