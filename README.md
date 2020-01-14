## ManjaroArm > ArchLinuxArm for your RPI

1. download the latest manjaro image for your raspberrypi and flash it with the balena-etcher on your usb external hdd/ssd
2. mount the drive into your computer and add Label tags for Boot and Root partitions
boot partition: LABEL=ROOT into cmdline.txt
```root=/dev/mmcblk0p2 -> root=LABEL=ROOT
```
root partition: LABEL=BOOT into /etc/fstab line
```/dev/mmcblk0p1 -> LABEL=BOOT
```
2. plug in your usb drive to your RaspberryPi
3. turn on
