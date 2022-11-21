# Home Assistant Operating System

Home Assistant Operating System (formerly HassOS) is a Linux based operating system optimized to host [Home Assistant](https://www.home-assistant.io) and its [Add-ons](https://www.home-assistant.io/addons/).

Home Assistant Operating System uses Docker as Container engine. It by default deploys the Home Assistant Supervisor as a container. Home Assistant Supervisor in turn uses the Docker container engine to control Home Assistant Core and Add-Ons in separate containers. Home Assistant Operating System is **not** based on a regular Linux distribution like Ubuntu. It is built using [Buildroot](https://buildroot.org/) and it is optimized to run Home Assistant. It targets single board compute (SBC) devices like the Raspberry Pi or ODROID but also supports x86-64 systems with UEFI.

## Features

- Lightweight and memory-efficient
- Minimized I/O
- Over The Air (OTA) updates
- Offline updates
- Modular using Docker container engine

## Supported hardware

- Raspberry Pi
- Hardkernel ODROID
- Asus Tinker Board
- Generic x86-64 (e.g. Intel NUC)
- Virtual appliances

See the full list and specific models [here](./Documentation/boards/README.md)

## Getting Started

If you just want to use Home Assistant the official [getting started guide](https://www.home-assistant.io/getting-started/) and [installation instructions](https://www.home-assistant.io/hassio/installation/) take you through how to download Home Assistant Operating System and get it running on your machine.

If you're interested in finding out more about Home Assistant Operating System and how it works read on...

## Development

If you don't have experience with embedded systems, Buildroot or the build process for Linux distributions it is recommended to read up on these topics first (e.g. [Bootlin](https://bootlin.com/docs/) has excellent resources).

The Home Assistant Operating System documentation can be found on the [Home Assistant Developer Docs website](https://developers.home-assistant.io/docs/operating-system).

### Components

- **Bootloader:**
  - [Barebox](https://barebox.org/) for devices that support UEFI
  - [U-Boot](https://www.denx.de/wiki/U-Boot) for devices that don't support UEFI
- **Operating System:**
  - [Buildroot](https://buildroot.org/) LTS Linux
- **File Systems:**
  - [SquashFS](https://www.kernel.org/doc/Documentation/filesystems/squashfs.txt) for read-only file systems (using LZ4 compression)
  - [ZRAM](https://www.kernel.org/doc/Documentation/blockdev/zram.txt) for `/tmp`, `/var` and swap (using LZ4 compression)
- **Container Platform:**
  - [Docker Engine](https://docs.docker.com/engine/) for running Home Assistant components in containers
- **Updates:**
  - [RAUC](https://rauc.io/) for Over The Air (OTA) and USB updates
- **Security:**
  - [AppArmor](https://apparmor.net/) Linux kernel security module

### Development builds

The Development build GitHub Action Workflow is a manually triggered workflow
which creates Home Assistant OS development builds. The development builds are
available at [os-builds.home-assistant.io](https://os-builds.home-assistant.io/).









#
#
#






# RK3308, RK3328, RK356X, RK3588 buildroot system

Now builds for rk3308, rk3328, RK3566 and RK3588

This repo generates a bootable sdcard image for the RK3xxx platform.
It is a 64 bit image. Based on buildroot, this directory is an external buildroot tree - it integrates into the main buildroot tree seamlessly.

For the RK3399 buildroot images, have a look at this dedicated repo : https://github.com/flatmax/buildroot.rk3399.external

# Initial setup

Clone buildroot. For example :

```
cd yourPath
git clone git://git.busybox.net/buildroot buildroot

# rock pi S tested with version : git checkout 2022.02.1
# rock pi 3a tested with version : git checkout 2022.08.1
# rock cm3 tested with version : git checkout 2022.05.3
# rock 5b tested with version : git checkout 2022.05
```

Make sure you have requirements :
```
sudo apt-get install -y build-essential gcc g++ autoconf automake libtool bison flex gettext
sudo apt-get install -y patch texinfo wget git gawk curl lzma bc quilt
```

If building in a minimal Docker image, you will also require :
```
sudo apt-get install -y cpio unzip rsync python3
```

***The above instructions apply to Debian-based distros.  Buildroot works on other distros, but installing the above dependencies is beyond the scope of this README; check your distro's package manager documentation.  Additionally the dash shell is required on distros where it is not the default.***

Clone the external buildroot tree :
```
git clone https://github.com/flatmax/buildroot.rockchip.git buildroot.rockchip.ext
```

# To make the system

```
# For the RockPi S
source buildroot.rockchip.ext/setup.rockPiS.sh yourPath/buildroot
# for the Radxa rock 3 a board
source buildroot.rockchip.ext/setup.rock3a.sh yourPath/buildroot
# for the Radxa rock 5b board
source buildroot.rockchip.ext/setup.rock5b.sh yourPath/buildroot
# for the Radxa rock cm3 io board
source buildroot.rockchip.ext/setup.cm3.sh yourPath/buildroot
# For the RockPi E (rk3328 based board) [needs more work]
source buildroot.rockchip.ext/setup.rockPiE.sh yourPath/buildroot
# For the Pine64 Quartz64 (rk3566 based board) [currently not working]
source buildroot.rockchip.ext/setup.quartz64.sh yourPath/buildroot
```

Make sure you have the buildroot downloads directory present (when you are in the yourPath/buildroot directory execute the following) :

```
mkdir ../buildroot.dl
```

# build the system

```
make
```

# installing

Insert your sdcard into your drive and make sure it isn't mounted. Write the image to the disk.

NOTE: The following command will overwrite any disk attached to $OF. Don't overwrite your root.

```
OF=/dev/sdf; rootDrive=`mount | grep " / " | grep $OF`; if [ -z $rootDrive ]; then sudo umount $OF[123456789]; sudo dd if=output/images/sdcard.img of=$OF; else echo you are trying to overwrite your root drive; fi
```

# using

Connect to the console debug uart with a serial cable. Or, add the openssh-server pacakge to the buildsystem, then ssh in as user root, no pass.

## ssh RSA keys

To use ssh, put your id_rsa.pub into the authorized_keys in the overlays directory. This will autoload your public RSA key to the embedded system so that you can login.
```
$ mkdir -p overlays/root/.ssh; chmod go-rwx overlays/root/.ssh
$ ls -ld overlays/root/.ssh
drwx------ 2 me me 4096 Aug  3  2016 overlays/root/.ssh
$ cat ~/.ssh/id_rsa.pub > overlays/root/.ssh/authorized_keys
$ ls -l overlays/root/.ssh/authorized_keys
-rw-r--r-- 1 me me 748 Feb 24 11:17 overlays/root/.ssh/authorized_keys
```

# TODO
## for the rk3308 board
Try to find suitable rock-chip boot binaries on github. rk3308_ddr_589MHz_uart0_m0_v1.26.bin can't be found in rkbin.
Shift uboot and the kernel to mainline Linux.
