# arm-linux-files

Various files needed to run Linux on Arm boards. The files are attached
to release [v1.0](https://github.com/xpack/arm-linux-files/releases/tag/qemu)

## Supported architectures

The supported architectures are:

- `armhf` for 32-bit devices
- `arm64` for 64-bit devices

As build environments, the Ubuntu 16.04.6 was selected, as it is the first
major release that supports 64-bit devices.

## Ubuntu 16.04.6 virtual disks

For convenience, ready to use virtual disk images are provided. The files
are pristine, as they resulted after the initial install, without any
changes. 
Due to GitHub limitations, the large files are split into several parts,
and must be reassembled after download.

Version 4.1.1 of QEMU was used, installed on macOS via Homebrew.

### The arm64 (64-bit) image

```console
curl -L --fail -o hda-ubu16-arm64.qcow2-aa https://github.com/xpack/arm-linux-files/releases/download/qemu/hda-ubu16-arm64.qcow2-aa
curl -L --fail -o hda-ubu16-arm64.qcow2-ab https://github.com/xpack/arm-linux-files/releases/download/qemu/hda-ubu16-arm64.qcow2-ab
curl -L --fail -o hda-ubu16-arm64.qcow2-ac https://github.com/xpack/arm-linux-files/releases/download/qemu/hda-ubu16-arm64.qcow2-ac
cat hda-ubu16-arm64.qcow2-aa hda-ubu16-arm64.qcow2-ab hda-ubu16-arm64.qcow2-ac >hda-ubu16-arm64.qcow2
```

For those who want to create this image themselves, below are the
steps used.

```console
mkdir -p qemu-arm
cd qemu-arm

curl -L --fail -o installer-ubu16-arm64-linux http://ports.ubuntu.com/ubuntu-ports/dists/xenial-updates/main/installer-arm64/current/images/netboot/ubuntu-installer/arm64/linux
curl -L --fail -o installer-ubu16-arm64-initrd.gz http://ports.ubuntu.com/ubuntu-ports/dists/xenial-updates/main/installer-arm64/current/images/netboot/ubuntu-installer/arm64/initrd.gz

qemu-img create -f qcow2 hda-ubu16-arm64.qcow2 32G

qemu-system-aarch64 -M virt -m 8G  -smp 4 -cpu cortex-a72 \
-kernel installer-ubu16-arm64-linux \
-initrd installer-ubu16-arm64-initrd.gz \
-drive if=none,file=hda-ubu16-arm64.qcow2,format=qcow2,id=hd \
-device virtio-blk-pci,drive=hd \
-netdev user,id=armnet \
-device virtio-net-pci,netdev=armnet \
-nographic -no-reboot
...
```

During the install, the default selections were used. You might want
to add your own user and configure local settings.

- Language: English
- Country: United States
- Hostname: `ubu16-arm64`
- Full name: Adminus Maximus
- Username: `primus/primus` <---
	- Use weak password: Yes
	- Encrypt home: No
- Time zone: Eastern
- Partition disk
	- Guided - use entire disk
	- Virtual disk 1 - 34.4 GB
	- #1 ext2, #2 ext4, #3 swap
	- Write changes: Yes
- Installation step failed during Select and install software
- Select and install software
- No automatic updates
- Software selection
	- standard system utilities
	- OpenSSH server
- Installation step failed during Select and install software
- Select and install software
- No automatic updates
- Software selection
	- standard system utilities
	- OpenSSH server
- Installation step failed  during Make the system bootable
- Continue without boot loader

```
No boot loader has been installed, either because you chose not to or 
because your specific architecture doesn't support a boot loader yet. 
                                                                      
You will need to boot manually with the /vmlinuz kernel on partition  
/dev/vda1 and root=/dev/vda2 passed as a kernel argument.             
```

- Is the system clock set to UTC: Yes
- Installation complete

As it can be seen, the procedure issued several errors, but retrying
managed to recover, execpt the step to install the bootloader, not
supported on Arm in this version, thus, when running the image under
QEMU, these files should be provided separatelly, as command line
options.

The files can be extracted from the qcow2 image, by mounting it with 
`qmu-nbd`, and the first partition as a regular filesystem, then the 
files are available for copying.


For 64-bit Arm Ubuntu 16.04.6, use:

- ubu16-arm64-initrd.img-4.4.0-170-generic
- ubu16-arm64-vmlinuz-4.4.0-170-generic

### The armhf (32-bit) image

For 32-bit Arm Ubuntu 16.04.6, use:

- ubu16-armhf-initrd.img-4.4.0-170-generic-lpae
- ubu16-armhf-vmlinuz-4.4.0-170-generic-lpae




