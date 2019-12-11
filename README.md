# arm-linux-files

Various files needed to run Linux on Arm boards. The QEMU related files 
are attached to release [qemu](https://github.com/xpack/arm-linux-files/releases/tag/qemu).

## Supported architectures

The supported Arm architectures are:

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

#### Download a ready to use image

```console
curl -L --fail -o ubu16-arm64-hda.qcow2-aa https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-arm64-hda.qcow2-aa
curl -L --fail -o ubu16-arm64-hda.qcow2-ab https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-arm64-hda.qcow2-ab
curl -L --fail -o ubu16-arm64-hda.qcow2-ac https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-arm64-hda.qcow2-ac
cat ubu16-arm64-hda.qcow2-aa ubu16-arm64-hda.qcow2-ab ubu16-arm64-hda.qcow2-ac >ubu16-arm64-hda.qcow2
```

##### How to prepare the image yourself

For those who want to create this image themselves, below are the
steps used.

```console
mkdir -p $HOME/Work/qemu-arm
cd $HOME/Work/qemu-arm

curl -L --fail -o ubu16-arm64-installer-linux http://ports.ubuntu.com/ubuntu-ports/dists/xenial-updates/main/installer-arm64/current/images/netboot/ubuntu-installer/arm64/linux
curl -L --fail -o ubu16-arm64-installer-initrd.gz http://ports.ubuntu.com/ubuntu-ports/dists/xenial-updates/main/installer-arm64/current/images/netboot/ubuntu-installer/arm64/initrd.gz

qemu-img create -f qcow2 ubu16-arm64-hda.qcow2 32G

qemu-system-aarch64 -M virt -m 8G  -smp 4 -cpu cortex-a72 \
-kernel ubu16-arm64-installer-linux \
-initrd ubu16-arm64-installer-initrd.gz \
-drive if=none,file=ubu16-arm64-hda.qcow2,format=qcow2,id=hd \
-device virtio-blk-pci,drive=hd \
-netdev user,id=armnet \
-device virtio-net-pci,netdev=armnet \
-nographic -no-reboot
...
```

During the install, the default selections were used. 

- Language: English
- Country: United States
- Hostname: `ubu16-arm64` <---
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
	- OpenSSH server <---
- Installation step failed during Select and install software
- Select and install software
- No automatic updates
- Software selection
	- standard system utilities
	- OpenSSH server <---
- Installation step failed during Make the system bootable
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
managed to recover, except the step to install the bootloader, not
supported on Arm in this version, thus, when running the image under
QEMU, the kernel and initrd files must be provided separatelly, as 
command line options.

#### Download the kernel and initrd

For 64-bit Arm Ubuntu 16.04.6, the files are:

- ubu16-arm64-initrd.img-4.4.0-170-generic
- ubu16-arm64-vmlinuz-4.4.0-170-generic

To download them, use:

```console
cd $HOME/Work/qemu-arm

curl -L --fail -o ubu16-arm64-initrd.img-4.4.0-170-generic https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-arm64-initrd.img-4.4.0-170-generic
curl -L --fail -o ubu16-arm64-vmlinuz-4.4.0-170-generic https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-arm64-vmlinuz-4.4.0-170-generic
```

##### How to extract the kernel and initrd yourself

For those who want to extract these files themselves, below are the
steps used.

The files can be extracted from the qcow2 image, by first mounting the
qcow2 file with `qmu-nbd`, then mounting the first partition as a regular 
filesystem, and finally copying the files.

```console
cd $HOME/Work/qemu-arm

sudo modprobe nbd max_part=8
sudo qemu-nbd --connect=/dev/nbd0 ubu16-arm64-hda.qcow2
sudo fdisk /dev/nbd0 -l
mkdir -p $HOME/tmp/mntpoint
sudo mount /dev/nbd0p1 $HOME/tmp/mntpoint
ls -l $HOME/tmp/mntpoint
cp $HOME/tmp/mntpoint/initrd.img-4.4.0-170-generic ubu16-arm64-initrd.img-4.4.0-170-generic
cp $HOME/tmp/mntpoint/vmlinuz-4.4.0-170-generic ubu16-arm64-vmlinuz-4.4.0-170-generic
sudo cp $HOME/tmp/mntpoint/vmlinuz-4.4.0-170-generic ubu16-arm64-vmlinuz-4.4.0-170-generic
sudo chown $(whoami) ubu16-arm64-vmlinuz-4.4.0-170-generic
sudo chmod +r ubu16-arm64-vmlinuz-4.4.0-170-generic
sudo chmod a-w ubu16-arm64-*
```

#### Start the virtual machine

With these new files available, it is possible to start the new virtual 
machine. If you work with a remote machine, preferably start qemu within
a `screen` session. To allow for remote access to the virtual machine, 
add a forwarder to the ssh port (for example via port 30064).

```console
screen -s qemu

cd $HOME/Work/qemu-arm

qemu-system-aarch64 -M virt -m 16G -smp 4 -cpu cortex-a72 \
-kernel ubu16-arm64-vmlinuz-4.4.0-170-generic \
-initrd ubu16-arm64-initrd.img-4.4.0-170-generic \
-append 'root=/dev/vda2' \
-drive if=none,file=ubu16-arm64-hda.qcow2,format=qcow2,id=hd \
-device virtio-blk-pci,drive=hd \
-netdev user,id=armnet,hostfwd=tcp::30064-:22 \
-device virtio-net-pci,netdev=armnet \
-nographic
...
```

Login as user primus (_primus_ is latin for _first_) to add your user
and configure local settings.

```console
sudo adduser ilg
sudo usermod -aG sudo ilg

sudo dpkg-reconfigure tzdata
cat /etc/timezone

sudo apt install -y screen
```

Now it is possible to login as the new user.

```console
ssh ilg@ilg-xbb-linux.local -p 30064
```

Preferably add the locale settings to the shell `.profile`:

```console
export LANGUAGE=en_US.UTF-8
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```

### The armhf (32-bit) image

For 32-bit Arm Ubuntu 16.04.6, use:

- ubu16-armhf-initrd.img-4.4.0-170-generic-lpae
- ubu16-armhf-vmlinuz-4.4.0-170-generic-lpae




