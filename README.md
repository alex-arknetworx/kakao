# Kakao Linux

Kakao is a Linux distribution for Lone Dynamics FPGA computers. At the moment it is essentially a buildroot configuration with extra packages that makes certain assumptions about the MicroSD card partition layout.

The goal of Kakao is to provide a minimal practical environment with access to useful timeless applications.

This repo contains everything necessary to build Kakao from source, the steps described here can replace the buildroot steps from the individual board repos.

Kakao can be configured to either:

  1. Keep a small rootfs in RAM and optionally mount additional filesystems later.
  2. Mount rootfs from an ext2 partition on the MicroSD card.

## Supported Boards

  - [Schoko](https://machdyne.com/product/schoko-computer)
  - [Konfekt](https://machdyne.com/product/konfekt-computer)
  - [Noir](https://machdyne.com/product/noir-computer)

Note: There is a [reward](https://machdyne.com/bounties/) for adding Linux support to our ICE40-based FPGA computers.

## Applications

Kakao includes the following notable applications:

  - vi
  - nano
  - tinyscheme
  - micropython
  - lesen (a script for reading books)
  - busybox (ls, cp, grep, awk, sed, etc.)

## Installation

### Preparing a MicroSD card

At least a 4GB MicroSD card is recommended. It is also recommended to create all 4 partitions even if they're not going to be used immediately.

Example partition layout for a 32GB MicroSD card:

| Partition # | Size | Mount Point | Purpose | Type | Id |
| ------------| ---- | ----------- | ------- | ---- | -- |
| 1 | 512MB | - | boot partition | FAT32 (LBA) | 0x0c |
| 2 | 4GB | / | rootfs or additional applications | ext2 | 0x83 |
| 3 | 24GB | /home | user data | ext2 | 0x83 |
| 4 | 1GB | /opt/ark | read-only data | ext2 | 0x83 |

Note: The 4th partition is mounted read-only and intended for our soon-to-be-released information distribution which contains public domain books and other useful data for offline computers.

#### Creating partitions

Determine the device for your MicroSD card and use it in place of /dev/sdX in the following instructions.

```
$ sudo fdisk -c /dev/sdX
```

Use the IDs in the above table when creating the partitions.

#### Formatting

```
$ sudo mkfs.msdos /dev/sdX1
$ sudo mkfs.ext2 /dev/sdX2
$ sudo mkfs.ext2 /dev/sdX3
$ sudo mkfs.ext2 /dev/sdX4
```

### Building Kakao Linux

```
$ git clone http://github.com/machdyne/kakao
$ git clone http://github.com/buildroot/buildroot
$ cd buildroot
$ make BR2_EXTERNAL=../kakao/buildroot/ kakao_defconfig
$ make
```

Copy the `Image` and `rootfs.cpio` files from output/images to the boot partition of the MicroSD card.

### Optional: Mount rootfs from the MicroSD card instead of using a ramdisk.

This option is generally slower but it frees 8MB of RAM and allows you to install more applications on the rootfs.

This requires modifying the bootargs option and then recompiling the device tree source file which can be found in the GitHub repo for your board (i.e. images/linux/konfekt.dts):

```
bootargs = "console=liteuart earlycon=liteuart,0xf0001000 rootwait root=/dev/mmcblk0p2 fbcon=logo-pos:0"
```

Remove the following lines to free the memory used by the RAM disk:

```
linux,initrd-start = <0x41000000>;
linux,initrd-end   = <0x41800000>;
```

Compile the dts file to dtb:

```
$ dtc -O dtb -o konfekt.dtb konfekt.dts
```

And then copy the DTB file to the boot directory of the MicroSD card.

To extract the rootfs onto the MicroSD card, make sure the second partition is formatted as ext2 and then:

```
$ sudo mount /dev/sdX2 /mnt
$ cd /mnt
$ sudo tar -xvf /path/to/buildroot/output/images/rootfs.tar
$ eject /dev/sdX2
```

# License

Kakao is a partial fork of [linux-on-litex-vexriscv](https://github.com/litex-hub/linux-on-litex-vexriscv) and this repo is released under the BSD 2-Clause License, with the following exception(s):

The tinyscheme buildroot package is from [buildroot-a13-olinuxino](https://github.com/m039/buildroot-a13-olinuxino) which uses the GPL-2.0 license. TinyScheme itself is licensed under a BSD-style license.
