## ArchLinuxArm for your RPI

### 1. download the latest Arch Linux Arm image for your raspberrypi and flash it with the balena-etcher on your usb external hdd/ssd
**.1.a if you prefer to boot via u-boot:
```
1. flash archlinux arm on an sd card and boot.
2. flash the usb with arch linux you prefer to boot
3. Add pcie_brcmstb into MODULES in /etc/mkinitcpio.conf - (both sd & usb)
4. Rebuild initrd with mkinitcpio -P - (both sd & usb)
OR create an image from sd card and you flash this image to usb...!!!! [easy way..]

```
**.1.b static ip with wpa_supplicant:
```
wpa_passphrase MYSSID passphrase > /etc/wpa_supplicant/wpa_supplicant-wlan0.conf
cat <<EOF | tee -a /etc/wpa_supplicant/wpa_supplicant-wlan0.conf
# Giving configuration update rights to wpa_cli
ctrl_interface=/run/wpa_supplicant
ctrl_interface_group=wheel
update_config=1
EOF
cat <<EOF | tee -a /etc/systemd/network/wlan0.network
[Match]
Name=wlan0

[Network]
DNSSEC=no
Address=192.168.1.150/24
Gateway=192.168.1.1
DNS=192.168.1.1
EOF
```
### 2. mount the drive into your computer and add Label tags for Boot and Root partitions

boot partition: LABEL=ROOT into cmdline.txt `root=/dev/mmcblk0p2 -> root=/dev/sda2`, /boot/cmdline.txt:
```
root=/dev/sda2 rw rootwait rootdelay=30 console=tty1 selinux=0 plymouth.enable=0 smsc95xx.turbo_mode=N dwc_otg.lpm_enable=0 elevator=noop audit=0 ipv6.disable=1
```
boot partition: /boot/config.txt:
```
initramfs initramfs-linux.img followkernel
hdmi_drive=2
dtparam=audio=on
disable_overscan=1
dtoverlay=gpio-ir,gpio_pin=24
dtparam=i2c_arm=on

#enable vc4
gpu_mem=320
dtoverlay=vc4-fkms-v3d
max_framebuffers=2
dtparam=i2c_arm=on
```
root partition: LABEL=BOOT into /etc/fstab line `/dev/mmcblk0p1 -> /dev/sda1`, /etc/fstab:
```
# Static information about the filesystems.
# See fstab(5) for details.

# <file system> <dir> <type> <options> <dump> <pass>
/dev/sda1  /boot   vfat    defaults        0       0
```
***Dont Resize the ROOT partition it will be automaticaly resized in the first boot!!***

this way the system always searches the drive with labeled BOOT & ROOT partitions with out problem and boot even if you add multiple usb disks.

### 3. Put the SD Card into your pi, power it on and login with alarm/alarm
First of all get root:
```
sudo arp-scan -l
ssh alarm@ip_address -p 22
su
```
Password for root: `root`.

### 4. Set timezone & enable systemd-timesyncd Service
```
ln -sf /usr/share/zoneinfo/Europe/Athens /etc/localtime
systemctl enable systemd-timesyncd
systemctl start systemd-timesyncd
echo 'EDITOR=nano' > /etc/environment
sed -i 's/#Color/Color/' /etc/pacman.conf
```
everytime RaspberryPi boots, given internet access, it will always sync hardware clock...

**5.A replace contents of `/etc/pacman.d/mirrorlist` with:**
```
echo '#Arch Linux RollbackMachine....' > /etc/pacman.d/mirrorlist
echo 'Server = http://tardis.tiny-vps.com/aarm/repos/2020/01/01/$arch/$repo' >> /etc/pacman.d/mirrorlist
```

**5.B update the system**
```
pacman-key --init
pacman-key --populate archlinuxarm
pacman -Syu --noconfirm
```
**5.C install optional software**
```
pacman -Suy --needed nfs-utils htop openssh btrfs-progs alsa-utils alsa-firmware alsa-lib alsa-plugins git zsh wget base-devel diffutils libnewt dialog wpa_supplicant iw crda lshw sudo i2c-tools lm_sensors uboot-tools samba v4l-utils cronie x11vnc neofetch pacman-contrib usbutils --noconfirm
```
**5.D install LightDM**
```
pacman -S xorg-server xf86-video-fbdev xorg-xrefresh lightdm-gtk-greeter --noconfirm
systemctl enable lightdm.service

cat > /etc/X11/xorg.conf.d/99-fbdev.conf <<- "EOF"
# This is a minimal sample config file, which can be copied to
# /etc/X11/xorg.conf in order to make the Xorg server pick up
# and load xf86-video-fbturbo driver installed in the system.
#
# When troubleshooting, check /var/log/Xorg.0.log for the debugging
# output and error messages.
#
# Run "man fbturbo" to get additional information about the extra
# configuration options for tuning the driver.

Section "Device"
  Identifier      "Allwinner A10/A13 FBDEV"
  Driver          "fbdev"
  Option          "fbdev" "/dev/fb0"
  Option          "SwapbuffersWait" "true"
EndSection
EOF
```
**5.D-1 install Mate-Desktop**
```
pacman -S mate mate-extra --noconfirm
```
**5.D-2 install XFCE-Desktop**
```
pacman -S xfce4 xfce4-goodies --noconfirm
```
**5.D-3 install Pantheon-Desktop**
```
pacman -S pantheon --noconfirm
```
### 6. Users & Hostname pi
```
hostnamectl set-hostname linux
useradd -d /home/linux -m -G wheel -s /bin/bash linux
groupadd -r autologin
gpasswd -a linux autologin
sed -i 's/# %wheel ALL=(ALL:ALL) NOPASSWD: ALL/%linux ALL=(ALL:ALL) NOPASSWD: ALL/' /etc/sudoers
sed -i 's/#autologin-user=/autologin-user=linux/' /etc/lightdm/lightdm.conf
sed -i 's/#autologin-user-timeout=0/autologin-user-timeout=0/' /etc/lightdm/lightdm.conf
sed -i 's/#user-session=default/user-session=kodi/' /etc/lightdm/lightdm.conf
passwd linux
```
reboot system and relogin to new user!
```
userdel -r alarm
```
### 7. Aur Helper - Trizen || Yay(optional)
```
cd /tmp
git clone https://aur.archlinux.org/trizen.git
cd trizen
makepkg -si --noconfirm
```
```
cd /tmp
pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si --noconfirm
```
### 8. Kodi
```
yay -S kodi-addon-inputstream-adaptive-any --noconfirm
pacman -S kodi-rpi --noconfirm
echo 'include kodi.config.txt' >> /boot/config.txt
#remove 'if (subject.user == "kodi")' from polkit rules file
sed -i '14d;2d' /usr/share/polkit-1/rules.d/10-kodi.rules
```

### 9. i2c interface
```
#enable i2c interface on kernel
echo 'dtparam=i2c_arm=on' >> /boot/config.txt
#enable modules on boot
echo 'i2c-dev' >> /etc/modules-load.d/i2c.conf
echo 'i2c-bcm2708' >> /etc/modules-load.d/i2c.conf
#give i2c interface non-root permissions
echo 'KERNEL=="i2c-[0-9]*", GROUP="wheel"' >> /etc/udev/rules.d/i2c.rules
#export device tree from sd
dtc -I fs -O dtb -o base.dtb /proc/device-tree
yay -S wd719x-firmware --noconfirm
yay -S kodi-addon-inputstream-adaptive-any --noconfirm
yay -S nano-syntax-highlighting --noconfirm
echo "include /usr/share/nano-syntax-highlighting/*.nanorc" >> /etc/nanorc
cp /etc/nanorc ~/.nanorc
```
### 10. Bashrc
```
#list
alias ls='ls --color=auto'
alias la='ls -a'

#fix obvious typo's
alias cd..='cd ..'
alias pdw="pwd"
alias pacu='trizen -Syu --noconfirm'

## Colorize the grep command output for ease of use (good for log files)##
alias grep='grep --color=auto'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'

#readable output
alias df='df -h'

#pacman unlock
alias unlock="sudo rm /var/lib/pacman/db.lck"

#free
alias free="free -mt"

#continue download
alias wget="wget -c"

#userlist
alias userlist="cut -d: -f1 /etc/passwd"

# Aliases for software managment
# pacman or pm
alias pacman='sudo pacman --color auto'
alias update='sudo pacman -Syyu --color auto'

#ps
alias ps="ps auxf"
alias psgrep="ps aux | grep -v grep | grep -i -e VSZ -e"

#add new fonts
alias fc='sudo fc-cache -fv'

#quickly kill conkies
alias kc='killall conky'

#hardware info --short
alias hw="hwinfo --short"

#skip integrity check
alias yayskip='yay -S --mflags --skipinteg'
alias trizenskip='trizen -S --skipinteg'

#Cleanup orphaned packages
alias cleanup='yay -Scc --noconfirm && sudo pacman -Rns $(pacman -Qtdq) --noconfirm && paccache -rk1'

#get the error messages from journalctl
alias jctl="journalctl -p 3 -xb"

#create a file called .bashrc-personal and put all your personal aliases
#in there. They will not be overwritten by skel.

[[ -f ~/.bashrc-personal ]] && . ~/.bashrc-personal

neofetch

```
finally if you install neofetch:
<img src="home.jpg">
