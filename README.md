## ManjaroArm > ArchLinuxArm for your RPI
this procedure counters the bug Kernel Panic on boot via latest kernel on Raspberry Pi 3+ and below.
https://archlinuxarm.org/forum/viewtopic.php?f=60&t=13821
https://github.com/raspberrypi/linux/issues/3087

1. download the latest manjaro image for your raspberrypi and flash it with the balena-etcher on your usb external hdd/ssd

```
wget https://osdn.net/projects/manjaro-arm/storage/rpi2/minimal/18.12.1/Manjaro-ARM-minimal-rpi2-18.12.1.img.xz```

2. mount the drive into your computer and add Label tags for Boot and Root partitions

boot partition: LABEL=ROOT into cmdline.txt
```
root=/dev/mmcblk0p2 -> root=LABEL=ROOT
```

root partition: LABEL=BOOT into /etc/fstab line
```
/dev/mmcblk0p1 -> LABEL=BOOT
```
this way the system always searches the drive with labeled BOOT & ROOT partitions with out problem and boot even if you add multiple usb disks.
2. plug in your usb drive to your RaspberryPi
3. turn on
