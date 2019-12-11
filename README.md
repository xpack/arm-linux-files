# arm-linux-files

This project includes various files needed to run Linux on Arm boards. The 
files are published as releases:

- [qemu](https://github.com/xpack/arm-linux-files/releases/tag/qemu) - 
files used to run Arm Linux on QEMU

## Supported architectures

The supported Arm architectures are:

- `armhf` for 32-bit devices
- `arm64` for 64-bit devices

As build environments, the Ubuntu 16.04.6 was selected, as it is the first
major release that supports 64-bit devices.

## Prerequisites

### A machine to run QEMU

Preferably a Linux machine, but macOS is also known to work.

In this case a headless Ubuntu 18.04 server Intel NUC with i7 and 32GB RAM 
was used (`ilg-xbb-linux.local`).

### QEMU 4.1.1

On Linux, the versions available on more conservative distributions,
like Ubuntu 18.04, are too old to be usable, and in this case it is
necessary to recompile it form sources.

In this case the separate folder need to be added to the PATH.

```console
PATH="$HOME/opt/qemu-4.1.1/bin:$PATH"
```

On some distributions there might be a separate `qemu-utils` which
needs to be installed.

On macOS, QEMU can be installed via Homebrew.

## Ubuntu 16.04.6 virtual disks

For convenience, ready to use virtual disk images are provided. The files
are pristine, as they resulted after the initial install, without any
changes. 
Due to GitHub limitations, the large files are split into several parts,
and must be reassembled after download.

### The arm64 (64-bit) image

#### Download a ready to use image

For 64-bit Arm Ubuntu 16.04.6, the image is:

- `ubu16-arm64-hda.qcow2`; it is about 2.4GB large and is assembled from 3 parts.

The commands to download and reassemble are:

```console
curl -L --fail -o ubu16-arm64-hda.qcow2-aa https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-arm64-hda.qcow2-aa
curl -L --fail -o ubu16-arm64-hda.qcow2-ab https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-arm64-hda.qcow2-ab
curl -L --fail -o ubu16-arm64-hda.qcow2-ac https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-arm64-hda.qcow2-ac
cat ubu16-arm64-hda.qcow2-aa ubu16-arm64-hda.qcow2-ab ubu16-arm64-hda.qcow2-ac >ubu16-arm64-hda.qcow2
```

##### How to prepare the image yourself

For those who want to create this image themselves, 
the steps used are documented in the separate
[HOW-TO-BUILD](https://github.com/xpack/arm-linux-files/blob/master/HOW-TO-BUILD.md)
file.

#### Download ready to use kernel and initrd

For 64-bit Arm Ubuntu 16.04.6, the ready to use files are:

- `ubu16-arm64-initrd.img-4.4.0-170-generic`
- `ubu16-arm64-vmlinuz-4.4.0-170-generic`

The commands to download them are:

```console
cd $HOME/Work/qemu-arm

curl -L --fail -o ubu16-arm64-initrd.img-4.4.0-170-generic https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-arm64-initrd.img-4.4.0-170-generic
curl -L --fail -o ubu16-arm64-vmlinuz-4.4.0-170-generic https://github.com/xpack/arm-linux-files/releases/download/qemu/ubu16-arm64-vmlinuz-4.4.0-170-generic
```

##### How to extract the kernel and initrd yourself

For those who want to extract these files themselves, 
the steps used are documented in the separate
[HOW-TO-BUILD](https://github.com/xpack/arm-linux-files/blob/master/HOW-TO-BUILD.md)
file.

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

[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Initializing cgroup subsys cpuset
[    0.000000] Initializing cgroup subsys cpu
[    0.000000] Initializing cgroup subsys cpuacct
[    0.000000] Linux version 4.4.0-170-generic (buildd@bos02-arm64-047) (gcc version 5.4.0 20160609 (Ubuntu/Linaro 5.4.0-6ubuntu1~16.04.12) ) #199-Ubuntu SMP Thu Nov 14 01:46:18 UTC 2019 (Ubuntu 4.4.0-170.199-generic 4.4.200)
[    0.000000] Boot CPU: AArch64 Processor [410fd083]
...

Ubuntu 16.04.6 LTS ubu16-arm64 ttyAMA0

ubu16-arm64 login: 
```

Login as user `primus` (_primus_ is latin for _first_). You can use
this initial user, or you, if you prefer, you can add your user
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

#### Power down

To power down the virtual machine, shutdown as usual:

```console
sudo poweroff
```

which is a shorcut for `shutdown -P now`.

In QEMU it is also possible to use Ctrl-A C, which will bring
the QEMU prompt, and issue the `system_powerdown` command.

### The armhf (32-bit) image

For 32-bit Arm Ubuntu 16.04.6, use:

- ubu16-armhf-initrd.img-4.4.0-170-generic-lpae
- ubu16-armhf-vmlinuz-4.4.0-170-generic-lpae

TODO: update with details.
