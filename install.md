# General Information
---
Arch linux keeps updating and if you want to keep up with the information, bookmark the website in given link below:
[Arch-home](https:/www.archlinux.org "Arch Homepage")
Some problems might occur which can occur to a particular device. Arch Wiki is well-documented and can be referred to resolve a issue.
[Arch-Wiki](https:/wiki.archlinux.org "Arch Wiki")
Forums can be a huge help if one cant resolve the issue
[Forum](https://bbs.archlinux.org/ "Arch forum")

# Booting the Installer
---
* Step 1 - Download the ISO for arch from the website or the link below
[Download](https://archlinux.org/download/)
* Step 2 - Make a flash drive using rofus or any other software.
* Step 2(alternative) - The iso can be booted directing from the OS

# Setting up a Network
---
Use the command below to know your network interfaces
```bash
ip addr show
```
To connect through wifi, use the following command and connect to your network.
```bash
wifi-menu
```
To check for an active internet connection, use the command
```bash
ping 8.8.8.8 -c 3
```
# Setting up the Disk
---
## non - UEFI Method
Use the command below to see all the available disks
```bash
lsblk
```
Target the right disk name
Use the partition disk cfdisk
```bash
cfdisk
```
Create a partition disk of type linux and write the partition table.

Now we are creating a LVM disk. Change the disk partition name based on your selection from the command below
```bash
pvcreate --dataalignment 1m /dev/sda1
```
Now we create a volume group for lvm
```bash
vgcreate volgroup0 /dev/sda1
```
The following commands create two logical volumes
```bash
lvcreate -L 35GB volgroup0 -n lv_root
lvcreate -l 100%FREE volgroup0 -n lv_home
```
To load the new volume group into the memory and activate the volume group
```bash
modprobe dm_mod
vgchange -ay
```
Format and mount the logical disks
```bash
mkfs.ext4 /dev/volgroup0/lv_root
mount /dev/volgroup0/lv_root /mnt
mkfs.ext4 /dev/volgroup0/lv_home
mkdir /mnt/home
mount /dev/volgroup0/lv_home /mnt/home
mkdir /mnt/etc
```
Save the partition layout
```bash
genfstab -U -p /mnt >> /mnt/etc/fstab
```
Check the file and see if everything is generated accordingly
```bash
cat /mnt/etc
```

## UEFI Method
Use the command below to see all the available disks
```bash
lsblk
```
Target the right disk name
Use the partition disk cfdisk
```bash
cfdisk
```
Create two partitions -
* /dev/sda1 - 500M (size) - EFI (type)
* /dev/sda2 - [desired size] - LVM (type)

Write the partition table and quit cfdisk

Format the EFI partition
```bash
mkfs.fat -F32 /dev/sda1
``` 
Now we create a volume group for lvm
```bash
vgcreate volgroup0 /dev/sda2
```
The following commands create two logical volumes
```bash
lvcreate -L 35GB volgroup0 -n lv_root
lvcreate -l 100%FREE volgroup0 -n lv_home
```
To load the new volume group into the memory and activate the volume group
```bash
modprobe dm_mod
vgchange -ay
```
Format and mount the logical disks
```bash
mkfs.ext4 /dev/volgroup0/lv_root
mount /dev/volgroup0/lv_root /mnt
mkfs.ext4 /dev/volgroup0/lv_home
mkdir /mnt/home
mount /dev/volgroup0/lv_home /mnt/home
mkdir /mnt/etc
```
Save the partition layout
```bash
genfstab -U -p /mnt >> /mnt/etc/fstab
```
Check the file and see if everything is generated accordingly
```bash
cat /mnt/etc
```

# Installing Arch Linux
---
Install the base packages
```bash
pacstrap -i /mnt base
```
Installation starts with the command
```bash
arch-chroot /mnt
```
Install the kernal
```bash
pacman -S linux-lts linux-lts-headers linux-firmware
```
Install the non-lts version for bleeding edge updates(alternative)
```bash
pacman -S linux linux-headers linux-firmware
```

Install a text editor, and a few developer packages
```bash
pacman -S nano vim base-devel openssh lvm2
```
Enable the openssh process
```bash
systemctl enable sshd
```
Install a few network packages
```bash
pacman -S networkmanager wpa_supplicant wireless_tools netctl dialog
```
Enable the Network Manager
```bash
systemctl enable NetworkManager
```
Change a line in the config file for the boot process to support
```bash
nano /etc/mkinitcpio.conf
```
Insert **lvm2** in the file
It should look similar to the line below
```vim
HOOKS=(base udev autodetect modconf block lvm2 filesystems keyboard fsck)
```
Run the following command to make the changes take effect
```bash
mkinitcpio -p linux-lts
```
If the non-lts kernal is installed then the command is as follows
```bash
mkinitcpio -p linux
```
Edit the locale file and uncomment the desired layout
```bash
nano /etc/locale.gen
```
Generate locale
```bash
locale-gen
```
Set the root password
```bash
passwd
```
Create a user
```bash
useradd -m -g users -G wheel [username]
```
Install the sudo package
```bash
pacman -S sudo
```
Associate the wheel group to sudo
```bash
visudo
```
Uncomment the line in the file
```vim
%wheel ALL=(ALL) ALL
```
Set the password for the user
```bash
passwd [username]
```
# Installing GRUB
---
Packages to install (**non -uefi method**)
```bash
pacman -S grub dosfstools os-prober mtools
```
Packages to install (**uefi method**)
```bash
pacman -S grub os-prober efibootmgr dosfstools
```
For **uefi method** only
```bash
mkdir /boot/EFI
mount /dev/sda1 /boot/EFI
```
To install GRUB (**non -uefi method**)
```bash
grub-install --target=i386-pc --recheck /dev/sda
```
To install GRUB (**uefi method**)
```bash
grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
```
Check if the locale directory is created. If not create one
```bash
ls -l /boot/grub
mkdir /boot/grub/locale [incase the directory is not there]
```
To display messages in English during the boot time, use the following command
```bash
cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
```
Generate the GRUB configuration file
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```
At this point, the PC can be rebooted.

# Post-install Tweaks
---
Creating the swap file
```bash
su
cd /
dd if=/dev/zero of=/swapfile bs=1M count=2048 status=progress
chmod 600 /swapfile
mkswap /swapfile
cp /etc/fstab /etc/fstab.bak
echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab
mount -a
swapon -a
```
Set the timezone
```bash
timedate set-timezone America/Costa_Rica
systemctl enable systemd-timesynced
```
Set the hostname
```bash
hostnamectl set-hostname [your_hostname]
```
Edit the /etc/hosts
```bash
nano /etc/hosts
```
Add the following lines in the file
```vim
127.0.0.1   localhost
127.0.1.1   [your_hostname]
```
Install micro code (intel)
```bash
sudo pacman -S intel-ucode
```
Install micro code (amd)
```bash
sudo pacman -S amd-ucode
```
Install xorg and a video driver
```bash
sudo pacman -S xorg [nvidia] [nvidia-lts] [mesa]
```
Now we have a working Arch machine.

You can now install your desired desktop environment.
