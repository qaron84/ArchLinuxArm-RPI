## ManjaroArm > ArchLinuxArm for your RPI
this procedure counters the bug Kernel Panic on boot via latest kernel on Raspberry Pi 3+ and below.
https://archlinuxarm.org/forum/viewtopic.php?f=60&t=13821
https://github.com/raspberrypi/linux/issues/3087

### 1. download the latest manjaro image for your raspberrypi and flash it with the balena-etcher on your usb external hdd/ssd

```
wget https://osdn.net/projects/manjaro-arm/storage/rpi2/minimal/18.12.1/Manjaro-ARM-minimal-rpi2-18.12.1.img.xz
```

### 2. mount the drive into your computer and add Label tags for Boot and Root partitions

boot partition: LABEL=ROOT into cmdline.txt `root=/dev/mmcblk0p2 -> root=LABEL=ROOT`, /boot/cmdline.txt:
```
root=LABEL=ROOT rw rootwait rootdelay=60 console=ttyAMA0,115200 console=tty1 selinux=0 plymouth.enable=0 smsc95xx.turbo_mode=N dwc_otg.lpm_enable=0 kgdboc=ttyAMA0,115200 elevator=noop audit=0
```

root partition: LABEL=BOOT into /etc/fstab line `/dev/mmcblk0p1 -> LABEL=BOOT`, /etc/fstab:
```
# Static information about the filesystems.
# See fstab(5) for details.

# <file system> <dir> <type> <options> <dump> <pass>
LABEL=BOOT  /boot   vfat    defaults        0       0
```
***Dont Resize the ROOT partition it will be automaticaly resized in the firts boot!!***

this way the system always searches the drive with labeled BOOT & ROOT partitions with out problem and boot even if you add multiple usb disks.

### 3. Put the SD Card into your pi, power it on and login with manjaro/manjaro
First of all get root:
```
sudo arp-scan -l
ssh manjaro@ip_address -p 22
su
```
Password for root: `root`.

### 4. Set timezone & enable systemd-timesyncd Service
```
ln -sf /usr/share/zoneinfo/Europe/Athens /etc/localtime
systemctl enable systemd-timesynced
systemctl start systemd-timesynced
```
everytime RaspberryPi boots, given internet access, it will always sync hardware clock...

### 5. System Update
**5.A replace contents of `/etc/pacman.conf` with:**
```
#
# /etc/pacman.conf
#
# See the pacman.conf(5) manpage for option and repository directives

#
# GENERAL OPTIONS
#
[options]
# The following paths are commented out with their default values listed.
# If you wish to use different paths, uncomment and update the paths.
#RootDir     = /
#DBPath      = /var/lib/pacman/
#CacheDir    = /var/cache/pacman/pkg/
#LogFile     = /var/log/pacman.log
#GPGDir      = /etc/pacman.d/gnupg/
HoldPkg      = pacman glibc
# If upgrades are available for these packages they will be asked for first
#XferCommand = /usr/bin/curl -C - -f %u > %o
#XferCommand = /usr/bin/wget --passive-ftp -c -O %o %u
#CleanMethod = KeepInstalled
#UseDelta    = 0.7
Architecture = armv7h

# Pacman won't upgrade packages listed in IgnorePkg and members of IgnoreGroup
IgnorePkg   = firmware-raspberrypi raspberrypi-bootloader raspberrypi-firmware
#IgnoreGroup =

#NoUpgrade   =
#NoExtract   =

# Misc options
#UseSyslog
Color
#TotalDownload
CheckSpace
VerbosePkgLists

# By default, pacman accepts packages signed by keys that its local keyring
# trusts (see pacman-key and its man page), as well as unsigned packages.
SigLevel    = Required DatabaseOptional
LocalFileSigLevel = Optional
#RemoteFileSigLevel = Required

# NOTE: You must run `pacman-key --init` before first using pacman; the local
# keyring can then be populated with the keys of all official Manjaro Linux
# packagers with `pacman-key --populate archlinuxarm manjaro-arm`.

#
# REPOSITORIES
#   - can be defined here or included from another file
#   - pacman will search repositories in the order defined here
#   - local/custom mirrors can be added here or in separate files
#   - repositories listed first will take precedence when packages
#     have identical names, regardless of version number
#   - URLs will have $repo replaced by the name of the current repo
#   - URLs will have $arch replaced by the name of the architecture
#
# Repository entries are of the format:
#       [repo-name]
#       Server = ServerName
#       Include = IncludePath
#
# The header [repo-name] is crucial - it must be present and
# uncommented to enable the repo.
#

[core]
Include = /etc/pacman.d/mirrorlist

[extra]
Include = /etc/pacman.d/mirrorlist

[community]
Include = /etc/pacman.d/mirrorlist

[alarm]
Include = /etc/pacman.d/mirrorlist

[aur]
Include = /etc/pacman.d/mirrorlist
# An example of a custom package repository.  See the pacman manpage for
# tips on creating your own repositories.
#[custom]
#SigLevel = Optional TrustAll
#Server = file:///home/custompkgs
```

**5.B replace contents of `/etc/pacman.d/mirrorlist` with:**
```
## Geo-IP based mirror selection and load balancing
Server = http://mirror.archlinuxarm.org/$arch/$repo
```
**5.C remove the file `/etc/lsb-release` with:**
`rm /etc/lsb-release`

**5.D update the system**
```
pacman-key --init
pacman-key --populate archlinuxarm
pacman -Syu --noconfirm
```
**5.E install optional software**
```
pacman -S --needed nfs-utils htop openssh autofs alsa-utils alsa-firmware alsa-lib alsa-plugins git zsh wget base-devel diffutils libnewt dialog wpa_supplicant wireless_tools iw crda lshw sudo i2c-tools lm_sensors
```
### 6. Users & Hostname
```
hostnamectl set-hostname your-hostname
sed -i 's/# %wheel ALL=(ALL) ALL/ %wheel ALL=(ALL) ALL/' /etc/sudoers
useradd -d /home/yourUserName -m -G wheel -s /bin/bash yourUserName
passwd yourUserName
sed -i 's/manjaro/yourUserName/' /etc/group
sed -i 's/yourUserName,yourUserName/yourUserName/' /etc/group
```
reboot system and relogin
### 7. Aur Helper - Trizen (optional)
```
cd /tmp
git clone https://aur.archlinux.org/trizen.git
cd trizen
makepkg -si
```
### 8. i2c interface
```
#enable i2c interface on kernel
echo 'dtparam=i2c_arm=on' >> /boot/config.txt
#enable modules on boot
echo 'i2c-dev' >> /etc/modules-load.d/i2c.conf
echo 'i2c-bcm2708' >> /etc/modules-load.d/i2c.conf
#give i2c interface non-root permissions
echo 'KERNEL=="i2c-[0-9]*", GROUP="wheel"' >> /etc/udev/rules.d/i2c.rules
```
reboot, re-login and check: `i2cdetect -y 0` or `i2cdetect -y 1`
