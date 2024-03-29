# arm-linux-files

This project includes various files needed to run Linux on Arm boards,
especially for development environments used to build Arm binaries.

The files are grouped by named releases:

- [qemu](https://github.com/xpack/arm-linux-files/releases/tag/qemu) - 
files used to run Arm Linux on QEMU

## Supported architectures

The supported Arm architectures are:

- `armhf` for 32-bit devices
- `arm64` for 64-bit devices

As development environments for building Arm binaries,
the **Ubuntu 16.04.6 LTS (xenial)** was selected, as it was the
first major release that supported 64-bit Arm devices.

At `uname -a` they identify as:

- `Linux ubu18-arm64 4.15.0-72-generic #81-Ubuntu SMP Tue Nov 26 12:21:09 UTC 2019 aarch64 aarch64 aarch64 GNU/Linux`
- `Linux ubu16-arm64 4.15.0-72-generic #81~16.04.1-Ubuntu SMP Tue Nov 26 16:31:09 UTC 2019 aarch64 aarch64 aarch64 GNU/Linux`
- `Linux ubu16-armhf 4.15.0-72-generic-lpae #81~16.04.1-Ubuntu SMP Tue Nov 26 19:06:09 UTC 2019 armv7l armv7l armv7l GNU/Linux`

Please note that the original kernel 4.4 that was distributed
with Ubuntu 16 is not stable and under 
heavy loads some aplications crash with strange errors,
like _Internal compiler error_.

## Prerequisites

### A machine to run QEMU

Preferably a Linux machine, but macOS is also known to work.

In this case a headless Ubuntu 18.04 server Intel NUC with i7 and 32GB RAM 
was used (`ilg-xbb-linux.local`).

### QEMU 4.1.1

On Linux, the QEMU versions available on more conservative distributions,
like Ubuntu 18.04, are too old to be usable, and in this case it is
necessary to recompile QEMU form sources.

In this case the separate folder must be added to the PATH.

```console
PATH="${HOME}/opt/qemu-4.1.1/bin:${PATH}"
```

or

```console
PATH="${HOME}/opt/homebrew/qemu/bin:${PATH}"
```

On some distributions there might be a separate `qemu-utils` which
needs to be installed with the package manager.

On macOS, QEMU can be installed via Homebrew.

### An install folder

Any place is fine, but somewhere in the HOME folder is better, for example:

```console
$ mkdir -p $HOME/opt/qemu-arm-vms
```

## Ubuntu cloud images

Ubuntu makes official images to be used in cloud environments,
and these images can also be used directly by QEMU.

- https://cloud-images.ubuntu.com/

This saves quite a lot of time to generate local instances of the 
images, and is the prefered method.

The Ubuntu cloud images have a default user called `ubuntu`,
but no password, and it is not possible to log to the machine.

The password must be passed as a separate user date file, created
with the following sequence

```console
cat >my-ubuntu-user-data-source <<EOF
#cloud-config
password: ubuntu
chpasswd: { expire: False }
ssh_pwauth: True
EOF
 
$ cloud-localds -v my-ubuntu-user-data.raw my-ubuntu-user-data-source
```

The `cloud-localds` tool can be installed on Ubuntu with:

```console
$ sudo apt install --yes cloud-utils
```

For convenience, this file can also be downloaded from the 
releases section.

There are other small trimmings that can be done after logging in to the virtual machine:

```console
$ touch .hushlogin
$ sudo touch /etc/cloud/cloud-init.disabled
```

## Ubuntu home-made virtual disks

Before cloud images was discoverd, it was common to create the
virtual images in house, but this process takes quite some time.

For convenience, ready to use virtual disk images are provided. The files
are pristine, as they resulted after the initial install, without any
changes. 
Due to GitHub limitations, the large files are split into several parts,
and must be reassembled after download.

### How to prepare the images yourself

For those who want to create the images themselves, 
the steps are documented in the separate
[HOW-TO-BUILD](https://github.com/xpack/arm-linux-files/blob/master/HOW-TO-BUILD.md)
file.

### How to extract the kernel and initrd yourself

For those who want to extract these files themselves, 
the steps are also documented in the separate
[HOW-TO-BUILD](https://github.com/xpack/arm-linux-files/blob/master/HOW-TO-BUILD.md)
file.

### Further customisations

These steps should be performed after the image is functional,
but are documented here, before all specific versions install
steps, since they apply to all versions.

The images, as they are published, are as generic as possible.
You can continue to use them as is, or further customise them.

For example you can continue to use the initial user, or, if 
you prefer, you can add your own user and configure local settings.

```console
$ sudo adduser ilg
$ sudo usermod -aG sudo ilg

$ sudo dpkg-reconfigure tzdata
$ cat /etc/timezone
```

Preferably add the locale settings to the shell `.profile`:

```console
export LANGUAGE=en_US.UTF-8
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```

For long tasks it is also recommended to start them inside a `screen` session:

```console
$ sudo apt install -y screen
```

#### Power down

To power down the virtual machine, shutdown as usual:

```console
$ sudo shutdown -P now
```

In QEMU it is also possible to use Ctrl-A C, which will bring
the QEMU prompt, and issue the `system_powerdown` command.

## Ubuntu virtual disks

### Ubuntu 18 arm64 (64-bit) cloud image

The Ubuntu system virtual image can be downloaded directly from 
Ubuntu.

Apart from the image itself, there are also several small files 
that need to be made available.

- the EFI raw image, is the monitor that identifies the kernel 
and the initrd and boots the system; it must be 64 MB long
- the EFI variable raw image, is the place where EFI can store 
different setting; it must be 64 MB long
- the cloud user data raw image, where different settings 
are passed to the generic cloud image (like passwords)
- the manifest file; it is not really needed by QEMU, it is 
downloaded for convenience, to know what packages are available

The first two must be passed to QEMU in this order as `pflash` drives.

```console
$ mkdir -p $HOME/opt-qemu-vms && cd $HOME/opt-qemu-vms

$ curl -L --fail -o ubu18-bionic-server-cloudimg-arm64.gcow2 https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-arm64.img
$ curl -L --fail -o ubu18-bionic-server-cloudimg-arm64.manifest https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-arm64.manifest

$ curl -L --fail -o aarch64-QEMU_EFI.fd http://snapshots.linaro.org/components/kernel/leg-virt-tianocore-edk2-upstream/latest/QEMU-AARCH64/RELEASE_GCC5/QEMU_EFI.fd

$ curl -L --fail -o my-ubuntu-user-data.raw https://github.com/xpack/arm-linux-files/releases/download/qemu/my-ubuntu-user-data.raw

$ dd if=/dev/zero of=aarch-qemu-efi.raw bs=1m count=64
$ dd if=aarch64-QEMU_EFI.fd of=aarch-qemu-efi.raw conv=notrunc

$ dd if=/dev/zero of=my-efi-vars.raw bs=1m count=64

$ qemu-system-aarch64 -cpu host -M virt -m 4G -smp 4 -cpu cortex-a72 \
-drive if=pflash,format=raw,file=aarch-qemu-efi.raw \
-drive if=pflash,format=raw,file=my-efi-vars.raw \
-drive if=none,format=qcow2,id=hd,file=ubu18-bionic-server-cloudimg-arm64.gcow2 \
-device virtio-blk-pci,drive=hd \
-drive format=raw,file=my-ubuntu-user-data.raw \
-netdev user,id=net0,hostfwd=tcp::30064-:22 \
-device virtio-net-pci,netdev=net0 \
-nographic

BdsDxe: failed to load Boot0001 "UEFI Misc Device" from VenHw(93E34C7E-B50E-11DF-9223-2443DFD72085,00): Not Found
BdsDxe: loading Boot0002 "UEFI Misc Device 2" from PciRoot(0x0)/Pci(0x1,0x0)
BdsDxe: starting Boot0002 "UEFI Misc Device 2" from PciRoot(0x0)/Pci(0x1,0x0)
error: no suitable video mode found.
error: no such device: root.

Press any key to continue...
[    2.120620] Couldn't get size: 0x800000000000000e
[    2.121182] MODSIGN: Couldn't get UEFI db list
[    2.121378] Couldn't get size: 0x800000000000000e
[    2.121588] Couldn't get size: 0x800000000000000e
-.mount
ufw.service
dev-hugepages.mount
dev-mqueue.mount
sys-kernel-debug.mount
...
systemd-user-sessions.service
systemd-logind.service
unattended-upgrades.service

Ubuntu 18.04.3 LTS ubuntu ttyAMA0

ubuntu login: ubuntu
Password: ubuntu
ubuntu@ubuntu:~$ sudo hostname ubu18-arm64
ubuntu@ubuntu:~$ sudo bash -c 'echo "ubu18-arm64" >/etc/hostname'
ubuntu@ubuntu:~$ touch .hushlogin
ubuntu@ubuntu:~$ sudo touch /etc/cloud/cloud-init.disabled
ubuntu@ubuntu:~$ exit
logout

Ubuntu 18.04.3 LTS ubu18-arm64 ttyAMA0

ubu18-arm64 login: 
Password: 
ubuntu@ubu18-arm64:~$
...
```

Subsequent restarts will be less verbose and do not start the 
cloud services.

```console
ubuntu@ubu18-arm64:~$ sudo shutdown -P now
```

With these options it is also possible to remotely login as 
user ubuntu via the port forwarder:

```console
$ ssh ubuntu@ilg-xbb-linux.local -p 30064
```

### Ubuntu 16 arm64 (64-bit) image

TODO: update to cloud image

#### Download a ready to use home-made image

For 64-bit Arm Ubuntu 16.04.6, the image is:

- `ubu16-arm64-hda.qcow2`; it is about 2.4GB large and must be 
assembled from 3 parts.

The commands to download and reassemble are:

```console
$ cd $HOME/Work/qemu-arm

$ curl -L --fail -o ubu16-arm64-hda.qcow2-aa https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-arm64-hda.qcow2-aa
$ curl -L --fail -o ubu16-arm64-hda.qcow2-ab https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-arm64-hda.qcow2-ab
$ curl -L --fail -o ubu16-arm64-hda.qcow2-ac https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-arm64-hda.qcow2-ac
$ chmod a-w ubu16-arm64-hda.qcow2-a*
$ cat ubu16-arm64-hda.qcow2-aa ubu16-arm64-hda.qcow2-ab ubu16-arm64-hda.qcow2-ac >ubu16-arm64-hda.qcow2
```

#### Download ready to use kernel and initrd

For 64-bit Arm Ubuntu 16.04.6, the ready to use files are:

- `ubu16-arm64-vmlinuz-4.15.0-72-generic`
- `ubu16-arm64-initrd.img-4.15.0-72-generic`

The commands to download them are:

```console
$ cd $HOME/Work/qemu-arm

$ curl -L --fail -o ubu16-arm64-vmlinuz-4.15.0-72-generic https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-arm64-vmlinuz-4.15.0-72-generic
$ curl -L --fail -o ubu16-arm64-initrd.img-4.15.0-72-generic https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-arm64-initrd.img-4.15.0-72-generic
$ chmod a-w ubu16-arm64-vmlinuz-4.15.0-72-generic ubu16-arm64-initrd.img-4.15.0-72-generic
```

#### Start the virtual machine

With these new files available, it is possible to start the new virtual 
machine. If you work with a remote machine, preferably start qemu within
a `screen` session. To allow for remote access to the virtual machine, 
add a forwarder to the ssh port (for example via port 30064).

After the system boots, you can login as user `primus` password `primus` 
(_primus_ is latin for _first_). 

```console
$ screen -S qemu

$ cd $HOME/Work/qemu-arm

$ qemu-system-aarch64 -M virt -m 8G -smp 4 -cpu cortex-a72 \
-kernel ubu16-arm64-vmlinuz-4.15.0-72-generic \
-initrd ubu16-arm64-initrd.img-4.15.0-72-generic \
-append 'root=/dev/vda2' \
-drive if=none,file=ubu16-arm64-hda.qcow2,format=qcow2,id=hd \
-device virtio-blk-pci,drive=hd \
-netdev user,id=armnet,hostfwd=tcp::30064-:22 \
-device virtio-net-pci,netdev=armnet \
-nographic

[    0.000000] Booting Linux on physical CPU 0x0000000000 [0x410fd083]
[    0.000000] Linux version 4.15.0-72-generic (buildd@bos02-arm64-055) (gcc version 5.4.0 20160609 (Ubuntu/Linaro 5.4.0-6ubuntu1~16.04.12)) #81~16.04.1-Ubuntu SMP Tue Nov 26 16:31:09 UTC 2019 (Ubuntu 4.15.0-72.81~16.04.1-generic 4.15.18)
[    0.000000] Machine model: linux,dummy-virt
...

Ubuntu 16.04.6 LTS ubu16-arm64 ttyAMA0

ubu16-arm64 login: 
Password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-72-generic aarch64)

...
primus@ubu16-arm64:~$ uname -a
Linux ubu16-arm64 4.15.0-72-generic #81~16.04.1-Ubuntu SMP Tue Nov 26 16:31:09 UTC 2019 aarch64 aarch64 aarch64 GNU/Linux
primus@ubu16-arm64:~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.6 LTS
Release:	16.04
Codename:	xenial
primus@ubu16-arm64:~$ lscpu
Architecture:          aarch64
Byte Order:            Little Endian
CPU(s):                4
On-line CPU(s) list:   0-3
Thread(s) per core:    1
Core(s) per socket:    4
Socket(s):             1
NUMA node(s):          1
NUMA node0 CPU(s):     0-3
primus@ubu16-arm64:~$ nproc
4
primus@ubu16-arm64:~$ sudo poweroff
[sudo] password for primus: 
         Stopping Session 1 of user primus.
[  OK  ] Closed Load/Save RF Kill Switch Status /dev/rfkill Watch.
...
[  OK  ] Reached target Shutdown.
[  207.517093] reboot: Power down

```

Now it is also possible to remotely login as the new user.

```console
$ ssh ilg@ilg-xbb-linux.local -p 30064
```

### The armhf (32-bit) image

TODO: update to cloud image

#### Download a ready to use home-made image

For 32-bit Arm Ubuntu 16.04.6, the image is:

- `ubu16-armhf-hda.qcow2`; it is about 2.4GB large and must be assembled from 3 parts.

The commands to download and reassemble are:

```console
$ cd $HOME/Work/qemu-arm

$ curl -L --fail -o ubu16-armhf-hda.qcow2-aa https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-armhf-hda.qcow2-aa
$ curl -L --fail -o ubu16-armhf-hda.qcow2-ab https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-armhf-hda.qcow2-ab
$ curl -L --fail -o ubu16-armhf-hda.qcow2-ac https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-armhf-hda.qcow2-ac
$ chmod a-w ubu16-armhf-hda.qcow2-a*
$ cat ubu16-armhf-hda.qcow2-aa ubu16-armhf-hda.qcow2-ab ubu16-armhf-hda.qcow2-ac >ubu16-armhf-hda.qcow2
```

#### Download ready to use kernel and initrd

For 32-bit Arm Ubuntu 16.04.6, the ready to use files are:

- `ubu16-armhf-vmlinuz-4.15.0-72-generic-lpae`
- `ubu16-armhf-initrd.img-4.15.0-72-generic-lpae`

The commands to download them are:

```console
$ cd $HOME/Work/qemu-arm

$ curl -L --fail -o ubu16-armhf-vmlinuz-4.15.0-72-generic-lpae https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-armhf-vmlinuz-4.15.0-72-generic-lpae
$ curl -L --fail -o ubu16-armhf-initrd.img-4.15.0-72-generic-lpae https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-armhf-initrd.img-4.15.0-72-generic-lpae
$ chmod a-w ubu16-armhf-vmlinuz-4.15.0-72-generic-lpae ubu16-armhf-initrd.img-4.15.0-72-generic-lpae
```

#### Start the virtual machine

With these new files available, it is possible to start the new virtual 
machine. If you work with a remote machine, preferably start qemu within
a `screen` session. To allow for remote access to the virtual machine, 
add a forwarder to the ssh port (for example via port 30064).

After the system boots, you can login as user `primus` password `primus` 
(_primus_ is latin for _first_). 

```console
$ screen -S qemu

$ cd $HOME/Work/qemu-arm

$ qemu-system-arm -M virt -m 8G -smp 4 -cpu cortex-a15 \
-kernel ubu16-armhf-vmlinuz-4.15.0-72-generic-lpae \
-initrd ubu16-armhf-initrd.img-4.15.0-72-generic-lpae \
-drive if=none,file=ubu16-armhf-hda.qcow2,format=qcow2,id=hd \
-device virtio-blk-device,drive=hd \
-netdev user,id=armnet,hostfwd=tcp::30032-:22 \
-device virtio-net-device,netdev=armnet \
-nographic -no-reboot

[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 4.15.0-72-generic-lpae (buildd@bos02-arm64-024) (gcc version 5.4.0 20160609 (Ubuntu/Linaro 5.4.0-6ubuntu1~16.04.12)) #81~16.04.1-Ubuntu SMP Tue Nov 26 19:06:09 UTC 2019 (Ubuntu 4.15.0-72.81~16.04.1-generic-lpae 4.15.18)
[    0.000000] CPU: ARMv7 Processor [412fc0f1] revision 1 (ARMv7), cr=30c5387d
[    0.000000] CPU: div instructions available: patching division code
[    0.000000] CPU: PIPT / VIPT nonaliasing data cache, PIPT instruction cache
[    0.000000] OF: fdt: Machine model: linux,dummy-virt
[    0.000000] Memory policy: Data cache writealloc
...

[  OK  ] Started Update UTMP about System Runlevel Changes.

Ubuntu 16.04.6 LTS ubu16-armhf ttyAMA0

ubu16-armhf login: primus
Password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-72-generic-lpae armv7l)

...
primus@ubu16-armhf:~$ uname -a
Linux ubu16-armhf 4.15.0-72-generic-lpae #81~16.04.1-Ubuntu SMP Tue Nov 26 19:06:09 UTC 2019 armv7l armv7l armv7l GNU/Linux
primus@ubu16-armhf:~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.6 LTS
Release:	16.04
Codename:	xenial
primus@ubu16-armhf:~$ lscpu
Architecture:          armv7l
Byte Order:            Little Endian
CPU(s):                4
On-line CPU(s) list:   0-3
Thread(s) per core:    1
Core(s) per socket:    4
Socket(s):             1
Model name:            ARMv7 Processor rev 1 (v7l)
primus@ubu16-armhf:~$ nproc
4
primus@ubu16-armhf:~$ sudo poweroff
[sudo] password for primus: 
[  OK  ] Stopped target Timers.
...
[  OK  ] Reached target Shutdown.
[  135.192446] reboot: Power down
```

Now it is also possible to remotely login as the new user.

```console
$ ssh ilg@ilg-xbb-linux.local -p 30032
```

## Comments

Although running a virtual machine sacrifices some performance, QEMU
is resonably fast, and for experimenting and even running some
build scripts, it is acceptable.

