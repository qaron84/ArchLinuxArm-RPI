### ManjaroArm > ArchLinuxArm for your RPI
this procedure counters the bug Kernel Panic on boot via latest kernel on Raspberry Pi 3+ and below.
https://archlinuxarm.org/forum/viewtopic.php?f=60&t=13821
https://github.com/raspberrypi/linux/issues/3087

## 1. download the latest manjaro image for your raspberrypi and flash it with the balena-etcher on your usb external hdd/ssd

```
wget https://osdn.net/projects/manjaro-arm/storage/rpi2/minimal/18.12.1/Manjaro-ARM-minimal-rpi2-18.12.1.img.xz
```

## 2. mount the drive into your computer and add Label tags for Boot and Root partitions

boot partition: LABEL=ROOT into cmdline.txt ```root=/dev/mmcblk0p2 -> root=LABEL=ROOT```, /boot/cmdline.txt:
```
root=LABEL=ROOT rw rootwait rootdelay=60 console=ttyAMA0,115200 console=tty1 selinux=0 plymouth.enable=0 smsc95xx.turbo_mode=N dwc_otg.lpm_enable=0 kgdboc=ttyAMA0,115200 elevator=noop audit=0
```

root partition: LABEL=BOOT into /etc/fstab line ```/dev/mmcblk0p1 -> LABEL=BOOT```, /etc/fstab:
```
# Static information about the filesystems.
# See fstab(5) for details.

# <file system> <dir> <type> <options> <dump> <pass>
LABEL=BOOT  /boot   vfat    defaults        0       0
```
***Dont Resize the ROOT partition it will be automaticaly resized in the firts boot!!***

this way the system always searches the drive with labeled BOOT & ROOT partitions with out problem and boot even if you add multiple usb disks.

## 3. Put the SD Card into your pi, power it on and login with manjaro/manjaro
```
sudo arp-scan -l
ssh manjaro@ip_address -p 22
su

```
3. turn on
