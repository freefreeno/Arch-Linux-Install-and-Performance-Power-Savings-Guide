# I would just get myself a spare USB and download a live linux system such as MX linux or something similar with gparted and just fix my partitions with that if you are new to Linux.

# Removing old boot entries BEFORE you install your new system while your still in your old system.
- Install efibootmgr
sudo pacman -S efibootmgr 

- List bootmgrs
sudo efibootmgr

- Mark boot entries active or inactive 
To set boot entries active:
sudo efibootmgr -b <bootnum> -a

To set inactive
sudo efibootmgr -b <bootnum> -A

- To delete boot entries
sudo efibootmgr -b <bootnum> -B

- Notice that Linux labels for nvme is /dev/nvme0n1, not /dev/sda.

# If you would like to clear the disk first use the wipe in the bios or use the expert mode in gdisk

gdisk /dev/nvme0n1 "or whatever your drive is called"
x
z
answer yes and enter

# Check the drives with:

parted -l

# Create gpt

parted /dev/nvme0n1 mklabel gpt

- Confirm the disk is gpt with 

parted /dev/nvme0n1 -l

# Create partitions

Create 260MiB esp,boot partition

mkpart primary fat32 1MiB 261MiB
260MiB

# Name the partition

name 1 'BOOT'

# Make it bootable

set 1 esp on

# Check EFI system partition.

unit MiB

print

# Create root "60GiB"

mkpart primary ext4 261MiB 61701MiB
61440MiB

# Name the root partition

name 3 'ROOT'

# Create swap partition

mkpart primary linux-swap 61701MiB 69893MiB
8192MiB

# Name the swap 

name 2 'SWAP'

# Create the home 

mkpart primary ext4 69893MiB 217349MiB
147,456MiB

# Name the home partition

name 4 'HOME'

# Check the work and alignment

Check alignment
align-check optimal 1
align-check optimal 2
align-check optimal 3
align-check optimal 4

print

# Save work 

quit

reboot to make sure all the partitions are recognized

# Activate swap

swapon /dev/nvme0n1p3

# List of the drives you should have so let us check with lsblk
/dev/nvme0n1p1 ESP
/dev/nvme0n1p2 ROOT
/dev/nvme0n1p3 SWAP
/dev/nvme0n1p4 HOME

# make sure we have internet connection

wifi-menu

# Update system clock

timedatectl set-ntp true
timedatectl status

# Synchronize the repos

pacman -Sy

## Mount Partitions ##

- Root always first

mount /dev/nvme0n1p2 /mnt

- Now mkdir for /boot and mount boot

mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot

- mkdir for /home and mount /home

mkdir /mnt/home
mount /dev/nvme0n1p4 /mnt/home

## get the fastest mirrors ##

pacman -S reflector

- Backup mirrorlist.

cp -r /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup

# Find the fastest mirrors with ONE of these commands. " I am giving you four to choose from so change your country if you use one of the last commands " 
reflector --latest 150 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist
reflector -c "United Sates" -f 12 -l 10 -n 12 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist
reflector -c 'United States' --latest 20 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist
reflector -c 'United States' --latest 200 --age 12 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist

pacman -Sy

# pacstrap Arch
pacstrap - i /mnt  base base-devel linux linux-firmware nano dosfstools dialog wireless_tools wpa_supplicant netctl man-db man-pages e2fsprogs usbutils intel-ucode bash-completion mtpfs acpid crda expac

# set country for crda
sed -i 's/#WIRELESS_REGDOM="US"/WIRELESS_REGDOM="US"/g' /etc/conf.d/wireless-regdom

or manually

nano /etc/conf.d/wireless-regdom

WIRELESS_REGDOM="US"

# Generate fstab
genfstab -U -p /mnt >> /mnt/etc/fstab

# Check and make sure your fstab file is correct
nano /mnt/etc/fstab

## Chroot into arch ##
arch-chroot /mnt

<2nd ssd>
# Set up auto mount for the second hard drive
# If you have two ssd's or HD's then this is how I do mine
# If you do not have two HD or ssd's then skip the steps with <2nd ssd> above them
# Make a directory to mount the dives
# Get the names of the drives
lsblk

<2nd ssd>
# Make the directory
mkdir /media

<2nd ssd>
# Permissions
chown $USER /media

<2nd ssd>
# Make separate directories to mount the three partitions
mkdir /media/themes
mkdir /media/storage
mkdir /media/backup

<2nd ssd>
# More Permissions
chown $USER /media/themes
chown $USER /media/storage
chown $USER /media/backup

<2nd ssd>
# If you had to you can change the the group also like this:
chown josh:users /media/backup

- and if you were doing this with a lot of folders already inside the directory then do it this way with the recursive flag
chown username:users /media/backup/ -R

<2nd ssd>
# Mount the partition
# Get names again
lsblk

mount /dev/nvme1n1p1 /media/themes
mount /dev/nvme1n1p2 /media/storage
mount /dev/nvme1n1p3 /media/backup

<2nd ssd>
# Mount these drives on boot
echo "UUID=$(lsblk -no UUID /dev/nvme1n1p1) /media/themes $(lsblk -no FSTYPE /dev/nvme1n1p1) defaults,noatime,nodev,nosuid,commit=30 0 2" >> /etc/fstab
echo "UUID=$(lsblk -no UUID /dev/nvme1n1p2) /media/storage $(lsblk -no FSTYPE /dev/nvme1n1p2) defaults,noatime,nodev,nosuid,commit=30 0 2" >> /etc/fstab
echo "UUID=$(lsblk -no UUID /dev/nvme1n1p3) /media/backup $(lsblk -no FSTYPE /dev/nvme1n1p3) defaults,noatime,nodev,nosuid,commit=30 0 2" >> /etc/fstab

<2nd ssd>
# Check the fstab after the first command and make sure it is correct before just hauling ass
# Also when done straighten the fstab up to your liking
# There is really no need to mount these drives now since they will be mounted at boot but if you want to you can do:
mount -a

## Language Uncomment en_US.UTF-8 for US english
sed -i 's/#en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/g' /etc/locale.gen

# Generate the locale
locale-gen

# Set it as your language
1  echo LANG=en_US.UTF-8 > /etc/locale.conf
2  export LANG=en_US.UTF-8

# Time
# If you do not know how to set this like I know I need America/Chicago then do
ln -sf /usr/share/zoneinfo/

hit tab and it will give you all the possibilities like America so then do

ln -sf /usr/share/zoneinfo/America/

and hit tab again to list all possibilities and once you have it then you can finish your link

ln -sf /usr/share/zoneinfo/America/Chicago  /etc/localtime

set hardware clock to utc
hwclock --systohc --utc

# Set hostname
echo archkde > /etc/hostname

# Configure hosts file

sudo nano /etc/hosts

add this:

127.0.0.1       localhost
::1             localhost
127.0.1.1       archkde.localdomain     archkde

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

# Update mirrors
pacman Syu

# Set password for root

passwd

# Add default user to the groups with

useradd -m -g users -G wheel -s /bin/bash username

# Set password for the user

passwd username

# Setting sudoers

EDITOR=nano visudo

If you want to be asked for password uncomment this

Uncomment the %wheel ALL=(ALL) ALL

but if your like me and don't care for the password prompt then uncomment the one right below this that says same thing without password

# Not reccomended but if you want to disable password for root then you can change 
root ALL=(ALL) ALL
to
root ALL=(ALL) NOPASSWD: ALL

save and close

# Install your bootloader " I use systemd boot since I only have the one OS "

bootctl --path=/boot install

# Create conf for boot loader

nano /boot/loader/entries/arch.conf

title Arch Linux
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img

save and close

#  Add PARTUUID of the ROOT partition to our boot loader configuration " My root is nvme0n1p2 but you can use lsblk to check to make sure "
# I also go ahead and add my kernel boot params here

echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/nvme0n1p2) rw nowatchdog i8042.nopnp quiet" >> /boot/loader/entries/arch.conf

# Check it to be sure 

nano /boot/loader/entries/arch.conf

It should be something like this:

title Arch Linux
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img
options root=PARTUUID=0e179967-fa6c-42b6-99e0-20a391354df2 rw nowatchdog i8042.nopnp quiet

# Change defaults for systemd boot
nano/boot/loader/loader.conf

timeout 2
console-mode max
editor   no

# Unmount

exit
umount -R /mnt
reboot

# Installing desktop and drivers
# Connect to wireless
sudo wifi-menu

# Add some needed repositories and uncomment color
Change #color
to
color

# Then add the repo that contains many good packages that would otherwise need to be m,anually built like Chromium vaapi
# https://github.com/archlinuxcn/repo
sudo nano /etc/pacman.conf
[archlinuxcn]
Server = https://mirror.xtom.com/archlinuxcn/$arch

# Add keys
sudo pacman -Syy && sudo pacman -S archlinuxcn-keyring

sudo pacman -Syu

# Rank mirrors again if downloads are slow

sudo cp -r /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup

# Rank the mirrors with one of these commands or a variant of your choosing
reflector --latest 150 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist
reflector -c "United Sates" -f 12 -l 10 -n 12 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist
reflector -c 'United States' --latest 20 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist
reflector -c 'United States' --latest 200 --age 12 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist

pacman -Syu

# Install xorg

- minimal install
sudo pacman -S xorg-server xorg-xrandr xorg-xdpyinfo xorg-xgamma

# Full group so not minimal
sudo pacman -S xorg
This should pull in xorg-server so pay attention

# Different #
# U can also start X auto style with:

sudo pacman -S xorg xorg-xinit  " for full"

or minimal

sudo pacman -S xorg-server xorg-xrandr xorg-xdpyinfo xorg-xgamma xorg-xinit

echo "exec startkde" > ~/.xinitrc

# Add multilib repos if you need 32 bit stuff but I do not use this

[multilib]
Include = /etc/pacman.d/mirrorlist

# Run an update 

sudo pacman -Syu

# Install drivers for X window system
# Info on the intel-compute-runtime and more stuff:
https://wiki.archlinux.org/index.php/GPGPU
https://wiki.archlinux.org/index.php/intel_graphics

# 32 and 64 bit #
sudo pacman -S mesa lib32-mesa intel-media-driver intel-media-sdk intel-compute-runtime lib32-vulkan-intel vulkan-intel lib32-vulkan-icd-loader vulkan-icd-loader ocl-icd lib32-ocl-icd libva-utils mesa-vdpau lib32-mesa-vdpau libva-mesa-driver lib32-libva-mesa-driver mesa-demos vdpauinfo xf86-video-fbdev xf86-video-vesa xf86-video-amdgpu vulkan-radeon lib32-vulkan-radeon ffmpeg-git xf86-video-intel xf86-video-ati intel-gpu-tools

# 64 bit #
sudo pacman -S mesa intel-media-driver intel-media-sdk intel-compute-runtime vulkan-intel vulkan-icd-loader ocl-icd libva-utils mesa-vdpau libva-mesa-driver mesa-demos vdpauinfo xf86-video-fbdev xf86-video-vesa xf86-video-amdgpu vulkan-radeon xf86-video-intel xf86-video-ati intel-gpu-tools

# Add a config file for your graphics to ensure ocl-icd loader is used but only do this if you have the ocl-icd installed.
sudo bash -c 'cat >/etc/ld.so.conf.d/00-usrlib.conf << EOF
/usr/lib
EOF'

# Create an Xorg configuration file
# First one is if you are only gonna use the modesetting driver
sudo nano /etc/X11/xorg.conf.d/20-modesetting.conf

Section "Device"
  Identifier "Intel Graphics"
  Driver "modesetting"
  Option "AccelMethod" "glamor"
  Option "DRI"         "3"
  Option      "Backlight"  "intel_backlight"
  BusID "PCI:0:2:0"
EndSection

# This is if you installed xf86-video-intel

sudo nano /etc/X11/xorg.conf.d/20-intel.conf

Section "Device"
    Identifier  "Intel Graphics"
    Driver      "intel"
    Option      "AccelMethod"    "sna"
    Option "DRI"         "3"
    Option      "Backlight"  "intel_backlight"
EndSection


Here is also a config example for AMD

Section "OutputClass"
	Identifier "AMDgpu"
	MatchDriver "amdgpu"
	Driver "amdgpu"
	BusID "PCI:0:1:0
EndSection


# Install display manager and Plasma desktop

sudo pacman -S sddm

sudo pacman -S plasma

- To select certain packages for installation:
Enter a selection (default=all): ^5-8 ^2
This selects all packages except 5 thru 8 and 2 for install
Excluding packages is achieved by prefixing a number or range of numbers with a caret (^).
Another example
Enter a selection (default=all): 1-15 20
In this case it will install packages 1 through 15 and package number 20.

# Install needed applications and tools 

sudo pacman -S dolphin kate konsole ark gwenview xdg-user-dirs spectacle git kompare gtk-engines ksystemlog sassc python-cairo rsync unrar alsa-utils fwupd nvme-cli extra-cmake-modules qbittorrent lostfiles mtpfs systemd-boot-pacman-hook chromium-vaapi efibootmgr pacmac-aur ufw svt-vp9 systemd-kcm

# Applications from the archlinuxcn repo that would otherwise have to be installed from the aur
systemd-boot-pacman-hook
chromium-vaapi pacmac-aur
systemd-kcm

## Enable firewall
sudo nano /etc/default/ufw
Then make sure "IPV6" is set to "yes", like so:
IPV6=yes

sudo ufw enable

# Set your enviroment variable for your intel-media-driver and disable QT logs

- sudo nano /etc/environment 

LIBVA_DRIVER_NAME=iHD
QT_LOGGING_RULES='*=false'

# Load huc and guc for full intel-media-driver fucntions " U can leave force_probe=* fastboot=1 off if it causes problems "

sudo bash -c 'cat > /etc/modprobe.d/i915.conf << EOF
 options i915 modeset=1 enable_guc=2 force_probe=* fastboot=1 
EOF'

# Enable early kms #

sudo sed -i 's/MODULES=()/MODULES=(i915)/g' /etc/mkinitcpio.conf

some machines will give errors if you do not load intel_agp before the i915 module so you can do

sudo sed -i 's/MODULES=()/MODULES=(intel_agp i915)/g' /etc/mkinitcpio.conf

sudo mkinitcpio -p linux

sudo bootctl update

# Enable sddm

sudo systemctl enable sddm.service

# Enable NetworkManager.service and systemd-resolved for dns cache
sudo systemctl enable NetworkManager.service

# Systemd resolved
# Enable systemd-resolved for dns cache 
sudo systemctl enable systemd-resolved.service

sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf

sudo bash -c 'cat > /etc/NetworkManager/conf.d/dns.conf << EOF
[main]
dns=systemd-resolved
EOF'

# CHECK YOUR WORK OR WHATEVER
# REBOOT ########################## REBOOT ###


# Download microsoft fonts 
https://github.com/fphoenix88888/ttf-mswin10-arch/blob/master/ttf-ms-win10-10.0.18362.116-1-any.pkg.tar.xz

Install with:
sudo pacman -U ttf-ms-win10-10.0.18362.116-1-any.pkg.tar.xz

# Add config file for chromium to set needed flags.
bash -c 'cat >~/.config/chromium-flags.conf << EOF
--flag-switches-begin --ignore-gpu-blacklist --enable-gpu-rasterization --enable-native-gpu-memory-buffers --enable-zero-copy --disable-gpu-driver-bug-workarounds --enable-quic --enable-parallel-downloading --enable-oop-rasterization --enable-smooth-scrolling --flag-switches-end
EOF'

# Open chromium sign in and then close it

# Confirm huc and guc
sudo cat /sys/kernel/debug/dri/0/i915_huc_load_status
- Should say something like this
HuC firmware: i915/kbl_huc_4.0.0.bin
        status: RUNNING
        version: wanted 4.0, found 4.0
        uCode: 225664 bytes
        RSA: 256 bytes

HuC status 0x00006080:

- Check i915_guc_load_status
sudo cat /sys/kernel/debug/dri/0/i915_guc_load_status
- Should say something like this
GuC firmware: i915/kbl_guc_33.0.0.bin
        status: RUNNING
        version: wanted 33.0, found 33.0
        uCode: 182528 bytes
        RSA: 256 bytes

GuC status 0x800330ed:
        Bootrom status = 0x76
        uKernel status = 0x30
        MIA Core status = 0x3

Scratch registers:
         0:     0xf0000000
         1:     0x81b000
         2:     0x40
         3:     0x0
         4:     0x4000
         5:     0x40
         6:     0x1002
         7:     0x0
         8:     0x0
         9:     0x0
        10:     0x0
        11:     0x0
        12:     0x0
        13:     0x0
        14:     0x0
        15:     0x0

# Confirm the iHD or i965 driver enviroment variables are Setting
set | grep LIBVA

- Should say:
LIBVA_DRIVER_NAME=iHD

# Confirm modeset #
sudo cat /sys/module/i915/parameters/modeset
You should get a 1 back

# Confirm intel media driver is working properly
vainfo
- Should show info like this
vainfo: VA-API version: 1.7 (libva 2.7.1)
vainfo: Driver version: Intel iHD driver for Intel(R) Gen Graphics - 20.1.1 ()
vainfo: Supported profile and entrypoints
      VAProfileNone                   : VAEntrypointVideoProc
      VAProfileNone                   : VAEntrypointStats
      VAProfileMPEG2Simple            : VAEntrypointVLD
      VAProfileMPEG2Simple            : VAEntrypointEncSlice
      VAProfileMPEG2Main              : VAEntrypointVLD
      VAProfileMPEG2Main              : VAEntrypointEncSlice
      VAProfileH264Main               : VAEntrypointVLD
      VAProfileH264Main               : VAEntrypointEncSlice
      VAProfileH264Main               : VAEntrypointFEI
      VAProfileH264Main               : VAEntrypointEncSliceLP
      VAProfileH264High               : VAEntrypointVLD
      VAProfileH264High               : VAEntrypointEncSlice
      VAProfileH264High               : VAEntrypointFEI
      VAProfileH264High               : VAEntrypointEncSliceLP
      VAProfileVC1Simple              : VAEntrypointVLD
      VAProfileVC1Main                : VAEntrypointVLD
      VAProfileVC1Advanced            : VAEntrypointVLD
      VAProfileJPEGBaseline           : VAEntrypointVLD
      VAProfileJPEGBaseline           : VAEntrypointEncPicture
      VAProfileH264ConstrainedBaseline: VAEntrypointVLD
      VAProfileH264ConstrainedBaseline: VAEntrypointEncSlice
      VAProfileH264ConstrainedBaseline: VAEntrypointFEI
      VAProfileH264ConstrainedBaseline: VAEntrypointEncSliceLP
      VAProfileVP8Version0_3          : VAEntrypointVLD
      VAProfileVP8Version0_3          : VAEntrypointEncSlice
      VAProfileHEVCMain               : VAEntrypointVLD
      VAProfileHEVCMain               : VAEntrypointEncSlice
      VAProfileHEVCMain               : VAEntrypointFEI
      VAProfileHEVCMain10             : VAEntrypointVLD
      VAProfileHEVCMain10             : VAEntrypointEncSlice
      VAProfileVP9Profile0            : VAEntrypointVLD
      VAProfileVP9Profile2            : VAEntrypointV

# Performance optimizations for 16 gb ram so if you have say 8 then change vm.dirty_ratio = 8 to vm.dirty_ratio = 16 and vm.dirty_background_ratio = 4 to vm.dirty_background_ratio = 8
sudo echo "echo \"vm.dirty_writeback_centisecs = 3000\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.dirty_expire_centisecs = 4500\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.dirty_ratio = 8\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.dirty_background_ratio = 4\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.swappiness = 5\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.vfs_cache_pressure = 50\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"net.ipv4.tcp_keepalive_time = 60\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"net.ipv4.tcp_keepalive_intvl = 10\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"net.ipv4.tcp_keepalive_probes = 6\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash

or just do
sudo echo "echo \"vm.dirty_writeback_centisecs = 3000\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.dirty_expire_centisecs = 4500\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.dirty_ratio = 8\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.dirty_background_ratio = 4\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.swappiness = 5\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.vfs_cache_pressure = 50\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash

# Create two tmpfs in fstab
tmpfs                   /tmp                                    tmpfs        noatime,nodev,nosuid          0 0
tmpfs                /home/josh/.cache/chromium/Default         tmpfs        noatime,nodev,nosuid          0 0

# Reboot 

# I use BFQ I/O scheduler
# set scheduler for NVMe
ACTION=="add|change", KERNEL=="nvme[0-9]n[0-9]", ATTR{queue/scheduler}="bfq"
# set scheduler for SSD and eMMC
ACTION=="add|change", KERNEL=="sd[a-z]|mmcblk[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="bfq"

- Or use 
# set scheduler for NVMe
ACTION=="add|change", KERNEL=="nvme[0-9]n[0-9]", ATTR{queue/scheduler}="none"
# set scheduler for SSD and eMMC
ACTION=="add|change", KERNEL=="sd[a-z]|mmcblk[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="bfq"

# disable wireless powersave
sudo bash -c 'cat > /etc/NetworkManager/conf.d/default-wifi-powersave.conf << EOF
[connection]
wifi.powersave = 2
EOF'

# Build packages in the tmpfs you just created ( This speeds up the building time of packages dramatically)
sudo nano /etc/makepkg.conf
uncomment MAKEFLAGS= and add it like this MAKEFLAGS="-j$(nproc)"
uncomment BUILDDIR=/tmp/makepkg
Change COMPRESSXZ=(xz -c -z -) to COMPRESSXZ=(xz -c -z - --threads=0)
also change where packages are kept after they are built by uncommenting 
PKGDEST=~/home/packages and make a folder in your home directory to store your packages. I changed mine to PKGDEST=~/.make-build/packages
- Don't forget to actually make the folder named ~/.make-build/packages in home directory.

# Remove packages and cleanup dependencies
sudo pacman -Rsc dhcpcd netctl dialog discover

sudo pacman -D --asdeps wpa_supplicant

# Install pamac-aur manually if you did not enable archlinuxcn repo which you needed to for Chromium-vaapi
git clone https://aur.archlinux.org/pamac-aur.git
cd pamac-aur
makepkg -sic
exit
sudo rm -r pamac-aur

# Install hooks 
- paccache-hook
- systemd-boot-hook if you did not install it above

## Make one hook manually
sudo mkdir /etc/pacman.d/hooks

sudo nano /etc/pacman.d/hooks/mirrorupgrade.hook
- Add this info

[Trigger]
Operation = Upgrade
Type = Package
Target = pacman-mirrorlist

[Action]
Description = Updating pacman-mirrorlist with reflector and removing pacnew...
When = PostTransaction
Depends = reflector
Exec = /bin/sh -c "reflector -c 'United States' --latest 100 --age 12 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist; rm -f /etc/pacman.d/mirrorlist.pacnew"


# Run a cleanup because there will be several packages left over at this point
sudo pacman -Sc && sudo pacman -Rns $(pacman -Qtdq)


# Set time and date automatically with KDE settings
Go to date and time and check the set automatically and hit apply

# Check the status of the ntp service with:
timedatectl status
timedatectl timesync-status

if not working then do:

sudo timedatectl set-ntp true 

# Enable trim once a week for ssd only
- Do not add discard-o to the fstab file

-sudo systemctl enable fstrim.timer

# Limit journal size
sudo nano /etc/systemd/journald.conf
systemmaxuse=200M
systemmaxfilesize=50M

# Disable pointing stick on laptop
sudo nano /etc/X11/xorg.conf.d/50-touchpad.conf
- Copy this inside but you may want to change things

Section "InputClass"
        Identifier "libinput pointer catchall"
        MatchIsPointer "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"
        Option "ignore" "on"  #Disable pointing stick
EndSection

Section "InputClass"
        Identifier "libinput touchpad catchall"
        MatchIsTouchpad "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"
        Option "Tapping" "on"
        Option "HorizontalScrolling" "false"
        Option "TappingButtonMap" "lrm"
        Option "TappingDrag" "on"
        Option "DisableWhileTyping" "on"
        Option "ScrollMethod" "twofinger"
EndSection

check after reboot

# Disable core dumps by creating this file
sudo nano /etc/systemd/coredump.conf

Storage=none
ProcessSizeMax=0

# 2nd method does not require reboot
sudo nano /etc/sysctl.d/50-coredump.conf
kernel.core_pattern=|/bin/false

To apply the settings without reboot run this command

sysctl -p /etc/sysctl.d/50-coredump.conf

# Disable Baloo file indexer for KDE
 balooctl suspend
 balooctl disable

# Power management
- Install thermald but do not enable it now.

sudo pacman -S acpica
git clone https://github.com/intel/dptfxtract.git
cd dptfxtract
sudo acpidump > acpi.out
acpixtract -a acpi.out
sudo ./dptfxtract *.dat

This will add your thermald config file to the proper location so then enable thermald with:
sudo systemctl enable --now thermald

# Getting detailed hardware info

Install
edid-decode-git
hw-probe

# Generic smartcard reader
# Install ccid from aur

# Check for intel-ucode early load
dmesg | grep microcode
journalctl -k --grep='microcode'

# Disable copy on select
right click the clipboard icon in panel and put check mark next to ignore selection

