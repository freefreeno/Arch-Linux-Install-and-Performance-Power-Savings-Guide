# Max performance for  ACTIVE-STATE POWER MANAGEMENT

# Disable hardware and software watchdogs
1. Sudo nano /boot/loader/entries/arch.conf
2. Add nowatchdog
3. sudo mkinitcpio -p linux
4. sudo bootctl update " Update grub if you use grub "

# Software watchdog just in case nowatchdog does not get it
sudo bash -c 'cat > /etc/sysctl.d/disable_watchdog.conf << EOF
 kernel.nmi_watchdog = 0
EOF'
3. Either reboot or load with sudo sysctl -p /etc/sysctl.d/disable_watchdog.conf

# Add performance config in sysctl.d " I have 16 GB of ram so adjust accordingly "
# Depending on amount of ram you will want to change vm.dirty_ratio and vm.dirty_background_ratio and also you can bump vm.dirty_writeback_centisecs = 3000 to vm.dirty_writeback_centisecs = 6000
# Do it all in one command
sudo echo "echo \"vm.dirty_writeback_centisecs = 3000\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.dirty_expire_centisecs = 4500\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.dirty_ratio = 8\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.dirty_background_ratio = 4\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.swappiness = 5\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.vfs_cache_pressure = 50\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash

# powertop reccomended
sudo echo "echo \"vm.dirty_writeback_centisecs = 1500\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.dirty_ratio = 8\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.dirty_background_ratio = 4\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.swappiness = 5\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash;sudo echo "echo \"vm.vfs_cache_pressure = 50\" >> /etc/sysctl.d/99-sysctl.conf" | sudo bash


# or add it manually

sudo nano /etc/sysctl.d/99-sysctl.conf

vm.dirty_writeback_centisecs = 3000
vm.dirty_expire_centisecs = 4500
vm.dirty_ratio = 8
vm.dirty_background_ratio = 4
vm.swappiness = 5
vm.vfs_cache_pressure = 50

Load the new rules with sysctl -p /etc/sysctl.d/99-sysctl.conf

# Match vm.dirty_writeback_centisecs = 3000 in your fstab or if you changed to 6000 or 1500 adjust accordingly
Add commit=30 to root and home like 
rw,noatime,commit=30

1. sudo nano /boot/loader/entries/arch.conf
2. Add pcie_aspm.policy=performance before quiet
3. sudo mkinitcpio -p linux
4. sudo bootctl update " Update grub if you use grub "
5. reboot to check that it worked
6. cat /sys/module/pcie_aspm/parameters/policy

# Max performance for sata link power management

sudo bash -c 'cat > /etc/udev/rules.d/ssd_power_save.rules << EOF
ACTION=="add", SUBSYSTEM=="scsi_host", KERNEL=="host*", ATTR{link_power_management_policy}="max_performance"
EOF'
3. Reboot
4. cat /sys/class/scsi_host/host*/link_power_management_policy

# Disable iwlwifi power_save
sudo bash -c 'cat > /etc/NetworkManager/conf.d/default-wifi-powersave.conf << EOF
[connection]
wifi.powersave = 2
EOF'

# Intel Graphics
sudo bash -c 'cat > /etc/modprobe.d/i915.conf << EOF
 options i915 modeset=1 enable_guc=2 force_probe=* fastboot=1 
EOF'

# Set IO schedulers
sudo bash -c 'cat > /etc/udev/rules.d/60-ioschedulers.rules << EOF
# set scheduler for NVMe
ACTION=="add|change", KERNEL=="nvme[0-9]n[0-9]", ATTR{queue/scheduler}="none"
# set scheduler for SSD and eMMC
ACTION=="add|change", KERNEL=="sd[a-z]|mmcblk[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="bfq"
EOF'

# Max power saving
# I disable the watchdogs no matter which method I am using so refer to the top

- https://wiki.archlinux.org/index.php/Power_management#PCI_Runtime_Power_Management
# To allow runtime power management only for devices that are known to work, use simple matching against vendor and device IDs (use lspci -nn to get these values):
# PCI Runtime Power Management
sudo bash -c 'cat > /etc/udev/rules.d/pci_pm.rules << EOF
ACTION=="add", SUBSYSTEM=="pci", ATTR{power/control}="auto"
EOF'

# Usb auto suspend
sudo bash -c 'cat > /etc/udev/rules.d/pci_pm.rules << EOF
ACTION=="add", SUBSYSTEM=="usb", ATTR{power/control}="auto"
EOF'


# Leave ACTIVE-STATE POWER MANAGEMENT at default and it will use the info in the Bios to obtain the rules
Should say default when checking with 
cat /sys/module/pcie_aspm/parameters/policy
[default]

# Balance performance for sata link power management

sudo bash -c 'cat > /etc/udev/rules.d/ssd_power_save.rules << EOF
ACTION=="add", SUBSYSTEM=="scsi_host", KERNEL=="host*", ATTR{link_power_management_policy}="med_power_with_dipm"
EOF'
3. Reboot
4. cat /sys/class/scsi_host/host*/link_power_management_policy


# Add config to turn sound power savings on
# https://wiki.archlinux.org/index.php/Power_management#Audio
sudo bash -c 'cat > /etc/modprobe.d/audio_powersave.conf << EOF
options snd_hda_intel power_save=1 power_save_controller=y
EOF'

if you have the ac97 replace options snd_hda_intel power_save=1 with :
options snd_ac97_codec power_save=1
in the command above

# You can use lsmod to see what is loaded.
I do lsmod > loaded_modules 
and then use konsole to search for the module name

# Blacklist unneeded modules 
# I use this which is a modified version of Kubuntu blacklist and the command needs a little fixing but just hit ctrl c if you get a blicking cursor and the file will still be there. " I will try and fix this "
sudo nano /etc/modprobe.d/blacklist.conf
- This file lists those modules which we don't want to be loaded by  alias expansion, usually so some other driver will be loaded for the device instead.
- evbug is a debug tool that should be loaded explicitly
blacklist evbug

- these drivers are very simple, the HID drivers are usually preferred
blacklist usbmouse
blacklist usbkbd

- replaced by e100
blacklist eepro100

- replaced by tulip
blacklist de4x5

- causes no end of confusion by creating unexpected network interfaces
blacklist eth1394

- snd_intel8x0m can interfere with snd_intel8x0, doesn't seem to support much
- hardware on its own (Ubuntu bug -2011, -6810)
blacklist snd_intel8x0m

- Conflicts with dvb driver (which is better for handling this device)
blacklist snd_aw2

- replaced by p54pci
blacklist prism54

- replaced by b43 and ssb.
blacklist bcm43xx

- most apps now use garmin usb driver directly (Ubuntu: -114565)
blacklist garmin_gps

- replaced by asus-laptop (Ubuntu: -184721)
blacklist asus_acpi

- low-quality, just noise when being used for sound playback, causes
- hangs at desktop session start (Ubuntu: -246969)
blacklist snd_pcsp

- ugly and loud noise, getting on everyone's nerves; this should be done by a
- nice pulseaudio bing (Ubuntu: -77010)
blacklist pcspkr

- EDAC driver for amd76x clashes with the agp driver preventing the aperture
- from being initialised (Ubuntu: -297750). Blacklist so that the driver
- continues to build and is installable for the few cases where its
- really needed.
blacklist amd76x_edac

- Blacklist watchdog modules for faster /boot/shutdown
blacklist iTCO_wdt
blacklist iTCO_vendor_support

- Disable webcam
blacklist uvcvideo

- Disable Bluetooth
blacklist bluetooth
blacklist hci
blacklist hci_uart
blacklist hci_vhci

- Disable joystick driver
blacklist joydev

- Disable the loading of the old radeon modules
blacklist radeon

- Some more reading
- Linux performance
http://www.brendangregg.com/linuxperf.html
