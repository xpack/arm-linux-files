# arm-linux-files

Various files needed to boot a Linux kernel on Arm boards.

The files are attached to release [v1.0](https://github.com/xpack/arm-linux-files/releases/tag/v1.0)

## Ubuntu 16.04.6 Arm kernel and initrd

When installing Ubuntu on QEMU, the installer announces that it could not install a bootloader:

```
No boot loader has been installed, either because you chose not to or 
because your specific architecture doesn't support a boot loader yet. 
                                                                      
You will need to boot manually with the /vmlinuz kernel on partition  
/dev/vda1 and root=/dev/vda2 passed as a kernel argument.             
```

As suggested, the solution is to provide the kernel and the initrd separately to QEMU, as command line options.

The files can be extracted from the qcow2 image, by mounting it with `qmu-nbd`, and the first partition as a regular filesystem, then the files are available for copying.

For 32-bit Arm Ubuntu 16.04.6, use:

- ubu16-armhf-initrd.img-4.4.0-170-generic-lpae
- ubu16-armhf-vmlinuz-4.4.0-170-generic-lpae

For 64-bit Arm Ubuntu 16.04.6, use:

- ubu16-arm64-initrd.img-4.4.0-170-generic
- ubu16-arm64-vmlinuz-4.4.0-170-generic


