# How to build

This file documents how the various files were created, and can be used 
as a template for creating custom images, for example larger ones.

## Ubuntu install default selections

During the install, for these reference images the default selections
were used. You can customise them for your own needs.

- Language: English
- Country: United States
- Hostname: `ubuNN-arm64` <--- (`ubu18-arm64`, `ubu16-arm64`; `ubu16-armhf` for 32-bit)
- Archive mirror country: United States <--- you may enter your favourite
	- us.ports.ubuntu.com
- HTTP proxy: (none)
- ... crunching silently for a few minutes, be patient ...
- Loading additional components
- ... crunching silently ...
- Full name: Primus Adminus
- Username: `primus/primus` <---
	- Use weak password: Yes
	- Encrypt home directory: No (present on old Ubuntu 16) 
- Time zone: Is this time zone correct? No, 
	- Select your time zone: Eastern <--- you may enter your location
- Partition disk
	- Partitioning method: Guided - use entire disk
	- Select disk to partition: Virtual disk 1 (vda) - 34.4 GB Virtio Block Device
	- The following partitions are going to be formatted:            
		- partition #1 of Virtual disk 1 (vda) as ESP            
		- partition #2 of Virtual disk 1 (vda) as ext4
	- Note: old Ubuntu 16 will list three partitions:
		- partition #1 of Virtual disk 1 (vda) as ext2
		- partition #2 of Virtual disk 1 (vda) as ext4
		- partition #5 of Virtual disk 1 (vda) as swap
	- Write the changes to disks: Yes
- Installing the base system... (this may take a while)
- [Installation step failed during: Select and install software (may not happen)]
- [Select and install software (retry this step)]
- Manage updates: No automatic updates
- Software selection
	- standard system utilities (not present in Ubuntu 18, default in Ubuntu 16)
	- OpenSSH server <---
	- Basic Ubuntu server <--- (not present in Ubuntu 16)
- Installation step failed during: Select and install software
- Select and install software (repeat until it completes this step)
- Manage updates: No automatic updates
- Software selection
	- standard system utilities
	- OpenSSH server <---
	- Basic Ubuntu server <--- (not present in Ubuntu 16)
- Is the system clock set to UTC: Yes
- Installation complete

As it can be seen, the procedure issued several errors, but retrying
manages to recover.

### Make the system bootable

When started with separate kernel and initrd, on Ubuntu 16, the install
also fails to write the bootloader:

- Installation step failed during: Make the system bootable
- Continue without boot loader

```
No boot loader has been installed, either because you chose not to or 
because your specific architecture doesn't support a boot loader yet. 
                                                                      
You will need to boot manually with the /vmlinuz kernel on partition  
/dev/vda1 and root=/dev/vda2 passed as a kernel argument.             
```


## QEMU EFI

To simplify usage, it is possible to configure QEMU to use an EFI monitor,
which will handle the boot steps, and there is no need to explicitly
specify the kernel and initrd files.

The solution is based on the 
[QEMU-EFI](http://snapshots.linaro.org/components/kernel/leg-virt-tianocore-edk2-upstream/latest/QEMU-AARCH64/)
from Linaro.

The solution is inspired from 
[Running an ISO installer image for arm64 (AArch64) using QEMU and KVM](http://www.redfelineninja.org.uk/daniel/2018/02/running-an-iso-installer-image-for-arm64-aarch64-using-qemu-and-kvm/)

## Separate kernel and initrd files

The initial way of starting QEMU was by passing explicit
files with the kernel and the initrd.

This method is simple to use, but requires extracting the 
two files from the image after the install completes.

It worked well with Ubuntu 16, and was inspired by:

- https://translatedcode.wordpress.com/2017/07/24/installing-debian-on-qemus-64-bit-arm-virt-board/
- https://translatedcode.wordpress.com/2016/11/03/installing-debian-on-qemus-32-bit-arm-virt-board/

However the install process with Ubuntu 18 or later failed
to create the initrd file in the `/boot` folder, and the more
automated EFI method was used.

## The arm64 (64-bit) images

### How to prepare the Ubuntu 18 virtual image file 

For 64-bit Arm Ubuntu 18.04, the ready to use file is:

- `ubu18-arm64-efi-hda.qcow2`

For those who want to create this image 
themselves, below are the steps used.
To get a more recent kernel, select the HWE folder.
The image size is 32GB, but can be easily changed to
any value.

```console
$ curl -L --fail -o aarch64-QEMU_EFI.img.gz http://snapshots.linaro.org/components/kernel/leg-virt-tianocore-edk2-upstream/latest/QEMU-AARCH64/RELEASE_GCC5/QEMU_EFI.img.gz
$ gunzip aarch64-QEMU_EFI.img.gz

$ curl -L --fail -o  ubu18-arm64-mini.iso http://ports.ubuntu.com/dists/bionic-updates/main/installer-arm64/current/images/hwe-netboot/mini.iso

$ qemu-img create -f qcow2 ubu18-arm64-efi-hda.qcow2 32G

$ qemu-system-aarch64 -cpu host -M virt -m 4G -smp 4 -cpu cortex-a72 \
-drive if=pflash,format=raw,file=aarch64-QEMU_EFI.img \
-drive if=none,format=qcow2,id=hd,file=ubu18-arm64-efi-hda.qcow2 \
-device virtio-blk-pci,drive=hd \
-drive if=none,format=raw,id=cd,file=ubu18-arm64-mini.iso \
-device virtio-blk-pci,drive=cd \
-netdev user,id=armnet \
-device virtio-net-pci,netdev=armnet \
-nographic \
-no-reboot \

[execution continues with the EFI monitor, which loads the kernel and starts the install...]

$ shasum -a 256 ubu18-arm64-efi-hda.qcow2
79b6138ccc49ba83cd7d726633d15cb5a8e8084c54753f7c6996e7d49df9497a

$ split -b 1024m ubu18-arm64-efi-hda.qcow2 ubu18-arm64-efi-hda.qcow2-
```

The result is:

```console
$ ls -l
total 15750240
-rw-r--r--  1 ilg  staff    67108864 Dec 18 18:07 aarch64-QEMU_EFI.img
-rw-r--r--  1 ilg  staff  3943497728 Dec 18 20:00 ubu18-arm64-efi-hda.qcow2
-rw-r--r--  1 ilg  staff  1073741824 Dec 18 20:01 ubu18-arm64-efi-hda.qcow2-aa
-rw-r--r--  1 ilg  staff  1073741824 Dec 18 20:01 ubu18-arm64-efi-hda.qcow2-ab
-rw-r--r--  1 ilg  staff  1073741824 Dec 18 20:01 ubu18-arm64-efi-hda.qcow2-ac
-rw-r--r--  1 ilg  staff   722272256 Dec 18 20:01 ubu18-arm64-efi-hda.qcow2-ad
-rw-r--r--  1 ilg  staff    64446464 Dec 18 18:07 ubu18-arm64-mini.iso
```

The 4 parts were published on GitHub.

The original page also mentions a `varstore`. It can be created as:

```console
$ qemu-img create -f qcow2 ubu18-arm64-efi-varstore.qcow2 64M
```

and added to QEMU with:

```
-drive if=pflash,format=qcow2,file=ubu18-arm64-efi-varstore.qcow2 \
```

(but don't ask what this is good for...)

### How to prepare the Ubuntu 16 virtual image file 

For 64-bit Arm Ubuntu 16.04.6, the ready to use file is:

- `ubu16-arm64-hda.qcow2`

For those who want to create this image 
themselves, below are the steps used.
To get a more recent kernel, select the HWE folder.
The image size is 32GB, but can be easily changed to
any value.

```console
$ cd $HOME/Work/qemu-arm

$ curl -L --fail -o ubu16-arm64-installer-linux http://ports.ubuntu.com/ubuntu-ports/dists/xenial-updates/main/installer-arm64/current/images/hwe-netboot/ubuntu-installer/arm64/linux
$ curl -L --fail -o ubu16-arm64-installer-initrd.gz http://ports.ubuntu.com/ubuntu-ports/dists/xenial-updates/main/installer-arm64/current/images/hwe-netboot/ubuntu-installer/arm64/initrd.gz

$ qemu-img create -f qcow2 ubu16-arm64-hda.qcow2 32G
Formatting 'ubu16-arm64-hda.qcow2', fmt=qcow2 size=34359738368 cluster_size=65536 lazy_refcounts=off refcount_bits=16

$ qemu-system-aarch64 -M virt -m 8G  -smp 4 -cpu cortex-a72 \
-kernel ubu16-arm64-installer-linux \
-initrd ubu16-arm64-installer-initrd.gz \
-drive if=none,file=ubu16-arm64-hda.qcow2,format=qcow2,id=hd \
-device virtio-blk-pci,drive=hd \
-netdev user,id=armnet \
-device virtio-net-pci,netdev=armnet \
-nographic -no-reboot

[    0.000000] Booting Linux on physical CPU 0x0000000000 [0x410fd083]
[    0.000000] Linux version 4.15.0-45-generic (buildd@bos02-arm64-015) (gcc version 5.4.0 20160609 (Ubuntu/Linaro 5.4.0-6ubuntu1~16.04.10)) #48~16.04.1-Ubuntu SMP Tue Jan 29 18:10:13 UTC 2019 (Ubuntu 4.15.0-45.48~16.04.1-generic 4.15.18)
[    0.000000] Machine model: linux,dummy-virt
...
```

During the install, the default selections were used. The steps are the
same as listed before, except the hostname, which is `ubu16-arm64`.

Compute the SHA 256 sum.

```
$ shasum -a 256 ubu16-arm64-hda.qcow2
3e02ed43a4c727e54ecf75e81170f1e776985864637cb23a15dc02916124e5cc
```

The resulting file is larger than 2 GB and must be split into separate
parts to be published on GitHub:

```console
$ split -b 1024m ubu16-arm64-hda.qcow2 ubu16-arm64-hda.qcow2-
```

### How to extract the kernel and initrd 

For 64-bit Arm Ubuntu 16.04.6, the ready to use files are:

- `ubu16-arm64-initrd.img-4.15.0-72-generic`
- `ubu16-arm64-vmlinuz-4.15.0-72-generic`

For those who want to extract these files themselves, below are the
steps used.

The files can be extracted from the image, by first mounting the
qcow2 file with `qmu-nbd`, then mounting the first partition as a regular 
filesystem, and finally copying the files.

```console
$ cd $HOME/Work/qemu-arm

$ sudo modprobe nbd max_part=8
$ sudo qemu-nbd --connect=/dev/nbd0 ubu16-arm64-hda.qcow2
$ sudo fdisk /dev/nbd0 -l
$ mkdir -p $HOME/tmp/mntpoint
$ sudo mount /dev/nbd0p1 $HOME/tmp/mntpoint
$ ls -l $HOME/tmp/mntpoint
$ sudo cp $HOME/tmp/mntpoint/vmlinuz-4.15.0-72-generic ubu16-arm64-vmlinuz-4.15.0-72-generic
$ cp $HOME/tmp/mntpoint/initrd.img-4.15.0-72-generic ubu16-arm64-initrd.img-4.15.0-72-generic
$ sudo chown $(whoami) ubu16-arm64-vmlinuz-4.15.0-72-generic
$ sudo chmod +r ubu16-arm64-vmlinuz-4.15.0-72-generic
$ sudo chmod a-w ubu16-arm64-vmlinuz-4.15.0-72-generic ubu16-arm64-initrd.img-4.15.0-72-generic
$ sudo umount $HOME/tmp/mntpoint
$ sudo qemu-nbd --disconnect /dev/nbd0
$ sudo rmmod nbd
```

## The armhf (32-bit) image(s)

### How to prepare the virtual image file 

For 32-bit Arm Ubuntu 16.04.6, the ready to use file is:

- `ubu16-armhf-hda.qcow2`

The SHA256 sum is `20c7cf917d1b42cb0597e53de55d9c17a831bc2817c49cb769176f9aa03e1613`.

For those who want to create this image 
themselves, below are the steps used.
To get a more recent kernel, select the HWE folder. 
The image size is 32GB, but can be easily changed to
any value.

```console
$ mkdir -p $HOME/Work/qemu-arm
$ cd $HOME/Work/qemu-arm

$ curl -L --fail -o ubu16-armhf-installer-vmlinuz http://ports.ubuntu.com/ubuntu-ports/dists/xenial-updates/main/installer-armhf/current/images/hwe-generic-lpae/netboot/vmlinuz
$ curl -L --fail -o ubu16-armhf-installer-initrd.gz http://ports.ubuntu.com/ubuntu-ports/dists/xenial-updates/main/installer-armhf/current/images/hwe-generic-lpae/netboot/initrd.gz

$ qemu-img create -f qcow2 ubu16-armhf-hda.qcow2 32G
Formatting 'ubu16-armhf-hda.qcow2', fmt=qcow2 size=34359738368 cluster_size=65536 lazy_refcounts=off refcount_bits=16

$ qemu-system-arm -M virt -m 8G -smp 4 -cpu cortex-a15 \
-kernel ubu16-armhf-installer-vmlinuz \
-initrd ubu16-armhf-installer-initrd.gz \
-drive if=none,file=ubu16-armhf-hda.qcow2,format=qcow2,id=hd \
-device virtio-blk-device,drive=hd \
-netdev user,id=armnet \
-device virtio-net-device,netdev=armnet \
-nographic -no-reboot

[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 4.15.0-45-generic-lpae (buildd@bos02-arm64-019) (gcc version 5.4.0 20160609 (Ubuntu/Linaro 5.4.0-6ubuntu1~16.04.10)) #48~16.04.1-Ubuntu SMP Tue Jan 29 19:55:21 UTC 2019 (Ubuntu 4.15.0-45.48~16.04.1-generic-lpae 4.15.18)
[    0.000000] CPU: ARMv7 Processor [412fc0f1] revision 1 (ARMv7), cr=30c5387d
[    0.000000] CPU: div instructions available: patching division code
[    0.000000] CPU: PIPT / VIPT nonaliasing data cache, PIPT instruction cache
[    0.000000] OF: fdt: Machine model: linux,dummy-virt
[    0.000000] Memory policy: Data cache writealloc
[    0.000000] efi: Getting EFI parameters from FDT:
[    0.000000] efi: UEFI not found.
[    0.000000] psci: probing for conduit method from DT.
[    0.000000] psci: PSCIv0.2 detected in firmware.
[    0.000000] psci: Using standard PSCI v0.2 function IDs
[    0.000000] psci: Trusted OS migration not required
[    0.000000] random: get_random_bytes called from start_kernel+0xb8/0x488 with crng_init=0
[    0.000000] percpu: Embedded 19 pages/cpu @(ptrval) s45580 r8192 d24052 u77824
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 2095424
[    0.000000] Kernel command line: 
[    0.000000] Dentry cache hash table entries: 131072 (order: 7, 524288 bytes)
[    0.000000] Inode-cache hash table entries: 65536 (order: 6, 262144 bytes)
[    0.000000] allocated 8388608 bytes of page_ext
[    0.000000] Memory: 8264324K/8388608K available (12288K kernel code, 919K rwdata, 4304K rodata, 2048K init, 740K bss, 124284K reserved, 0K cma-reserved, 7602176K highmem)
...
```

During the install, the default selections were used. The steps are the
same as for the 64-bit image, except the hostname, which is `ubu16-armhf`.

The command to split the file is:

```console
$ split -b 1024m ubu16-armhf-hda.qcow2 ubu16-armhf-hda.qcow2-
```

### How to extract the kernel and initrd 

For 32-bit Arm Ubuntu 16.04.6, the ready to use files are:

- `ubu16-armhf-vmlinuz-4.15.0-72-generic-lpae`
- `ubu16-armhf-initrd.img-4.15.0-72-generic-lpae`

For those who want to extract these files themselves, below are the
steps used.

The files can be extracted from the image, by first mounting the
qcow2 file with `qmu-nbd`, then mounting the first partition as a regular 
filesystem, and finally copying the files.

```console
$ cd $HOME/Work/qemu-arm

$ sudo modprobe nbd max_part=8
$ sudo qemu-nbd --connect=/dev/nbd0 ubu16-armhf-hda.qcow2
$ sudo fdisk /dev/nbd0 -l
$ mkdir -p $HOME/tmp/mntpoint
$ sudo mount /dev/nbd0p1 $HOME/tmp/mntpoint
$ ls -l $HOME/tmp/mntpoint
$ sudo cp $HOME/tmp/mntpoint/vmlinuz-4.15.0-72-generic-lpae ubu16-armhf-vmlinuz-4.15.0-72-generic-lpae
$ cp $HOME/tmp/mntpoint/initrd.img-4.15.0-72-generic-lpae ubu16-armhf-initrd.img-4.15.0-72-generic-lpae
$ sudo chown $(whoami) ubu16-armhf-vmlinuz-4.15.0-72-generic-lpae
$ sudo chmod +r ubu16-armhf-vmlinuz-4.15.0-72-generic-lpae
$ sudo chmod a-w ubu16-armhf-vmlinuz-4.15.0-72-generic-lpae ubu16-armhf-initrd.img-4.15.0-72-generic-lpae
$ sudo umount $HOME/tmp/mntpoint
$ sudo qemu-nbd --disconnect /dev/nbd0
$ sudo rmmod nbd
```
