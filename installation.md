# Installing Arch Linux
Reference: https://wiki.archlinux.org/title/Installation_guide

## End Goal & Setup Details
- Install on physical hardware (with cpu ucodes)
- UEFI Boot
- GPT Partition Tables
- Encrypted Drives
- Systemd bootloader
- Swapfile (instead of swap partition)
- KDE Plasma Desktop
- Optional wifi setup

## Process:

### 0. Download ISO, Flash USB
- Downlaod the ISO using a straight download or torrent method: https://archlinux.org/download/
- Copy the SHA256 key and paste into Notepad (if on Windows) or Linux text editor
- On Windows, use PowerShell to confirm the signature and that your ISO is complete and hasn't been tampered with.
```
Get-FileHash .\archlinux-2023.08.01-x86_64.iso -Algorithm SHA256
```
```
Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
SHA256          3BF1287333DE5C26663B70A17CE7573F15DC60780B140CBBD1C720338C0ABAC5       C:\Users\...
```
- On Linux
```
sha256sum archlinux-2023.08.01-x86_64.iso
```

### 1. First check internet connectivity with `ping`
```
root@archiso~# ping google.com
```

- To setup wifi follow this guide for iwtcl: https://wiki.archlinux.org/title/Iwd#iwctl
```
root@archiso~# iwctl
```
```
[iwd]# device list
[iwd]# device <device> set-property Powered on
[iwd]# adapter <adapter> set-property Powered on
[iwd]# station <device> scan
[iwd]# station <device> get-networks
[iwd]# station <device> connect SSID
```
- or use this command to connect:
```
iwctl --passphrase <passphrase> station <device> connect <SSID>
```


### 2. Set Font, Keymap, and Time
- Load keyboard keymap. Default is US.
- First, you can list the avaiable keymaps already available in the ISO image and wildcard search using asterisks
- If searching for US based ones, the following command searches only those with `us` in the name and ending in `.map.gz`
```
root@archiso~# ls /usr/share/kbd/keymaps/**/*us*.map.gz
```
- Select the keyboard keymap with `loadkeys` command:
```
root@archiso~# loadkeys us
```
- Check platform size (if 64 is shown, your CPU platform is 64-bit)
```
root@archiso~# cat /sys/firmware/efi/fw_platform_size
64
```
- Setfont if needed:
```
root@archiso~# setfont ter-d32n
```
- Check to ensure your motherboard and cpu support UEFI (this can also be confirmed with the previous step)
- Setup timezone and time/date information (this is only for the current live boot, we will redo this permanently later for the OS)
- Use the command `timedatectl` to check and set time information: https://man.archlinux.org/man/timedatectl.1
```
root@archiso~# timedatectl set-timezone America/Chicago
```
```
root@archiso~# timedatectl set-ntp true
```


### 3. Disk Setup
```
root@archiso~# lsblk
NAME    MAJ:MIN RM  SIZE    RO  TYPE    MOUNTPOINTS
loop0   7:0     0   693.4M  1   loop    /run/archiso/airootfs
sda     8:0     0   100G    0   disk    
sr0     11:0    1   813.3M  0   rom     /run/archiso/bootmr
```
- Wipe disk you want to install on to start fresh with `gdisk`
```
root@archiso~# gdisk /dev/sda
```
- Interactive Actions: {
expert mode (x), 
zap (z), 
yes
}


### 4. Section 1.9 of Installation Guide - Partition the disks

- Check disks available with `lsblk`
```
root@archiso~# lsblk
NAME    MAJ:MIN RM  SIZE    RO  TYPE    MOUNTPOINTS
loop0   7:0     0   693.4M  1   loop    /run/archiso/airootfs
sda     8:0     0   100G    0   disk    
sr0     11:0    1   813.3M  0   rom     /run/archiso/bootmnt
```
- Enter interactive session with `cgdisk` (note: swap is optional)
```
root@archiso~# cgdisk /dev/sda
```
- Interactive Actions: {
    create three partitions: boot (EF00), swap (8200), root (8300),
    write,
    quit
- Boot partition uses `EF00` filesystem type: https://wiki.archlinux.org/title/EFI_system_partition
- Check disks again with `lsblk` to ensure all partitions were created
```
root@archiso~# lsblk
NAME    MAJ:MIN RM  SIZE    RO  TYPE    MOUNTPOINTS
loop0   7:0     0   693.4M  1   loop    /run/archiso/airootfs
sda     8:0     0   100G    0   disk    
|-sda1  8:1     0   1G      0   part
|-sda2  8:2     0   8G      0   part
|-sda3  8:3     0   91G     0   part
sr0     11:0    1   813.3M  0   rom     /run/archiso/bootmnt
```

### 6. Section 1.10/11 of Installation Guide
#### Reference: https://wiki.archlinux.org/title/File_systems

#### 6.1 Optional encyrpted partition with LUKS: https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system#LUKS_on_a_partition
```
root@archiso ~ # lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0         7:0    0   673M  1 loop /run/archiso/airootfs
sda           8:0    1  14.5G  0 disk
└─sda1        8:1    1  14.5G  0 part
nvme0n1     259:0    0 465.8G  0 disk
├─nvme0n1p1 259:1    0     1G  0 part
└─nvme0n1p2 259:2    0 464.8G  0 part
```
- Using nvme0n1p2 as root partition, use the `cryptsetup` command with optinos `-y` to verify password and `-v` for verbose
```
root@archiso ~ # cryptsetup -y -v luksFormat /dev/nvme0n1p2
WARNING: Device /dev/nvme0n1p2 already contains a 'swap' superblock signature.

WARNING!
========
This will overwrite data on /dev/nvme0n1p2 irrevocably.

Are you sure? (Type 'yes' in capital letters): YES
Enter passphrase for /dev/nvme0n1p2:
Verify passphrase:
Existing 'swap' superblock signature on device /dev/nvme0n1p2 will be wiped.
Key slot 0 created.
Command successful.
cryptsetup -y -v luksFormat /dev/nvme0n1p2  18.43s user 0.49s system 103% cpu 18.220 total
```
- Open the encrypted partition, create the filesystem, and mount the partition properly:
```
root@archiso ~ # cryptsetup open /dev/nvme0n1p2 root
Enter passphrase for /dev/nvme0n1p2:
cryptsetup open /dev/nvme0n1p2 root  7.17s user 0.19s system 131% cpu 5.595 total
root@archiso ~ # mkfs.ext4 /dev/mapper/root
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 121830144 4k blocks and 30457856 inodes
Filesystem UUID: bf4342bd-52ca-470d-aca0-2164f829ea90
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
        102400000

Allocating group tables: done
Writing inode tables: done
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done

root@archiso ~ # mount /dev/mapper/root /mnt
root@archiso ~ # lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
loop0         7:0    0   673M  1 loop  /run/archiso/airootfs
sda           8:0    1  14.5G  0 disk
└─sda1        8:1    1  14.5G  0 part
nvme0n1     259:0    0 465.8G  0 disk
├─nvme0n1p1 259:1    0     1G  0 part
└─nvme0n1p2 259:2    0 464.8G  0 part
  └─root    254:0    0 464.7G  0 crypt /mnt
```
- Verify everything is working by closing the partition and unmounting, then redoing both:
```
root@archiso ~ # umount /mnt
root@archiso ~ # cryptsetup close root
root@archiso ~ # lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0         7:0    0   673M  1 loop /run/archiso/airootfs
sda           8:0    1  14.5G  0 disk
└─sda1        8:1    1  14.5G  0 part
nvme0n1     259:0    0 465.8G  0 disk
├─nvme0n1p1 259:1    0     1G  0 part
└─nvme0n1p2 259:2    0 464.8G  0 part
root@archiso ~ # cryptsetup open /dev/nvme0n1p2 root
Enter passphrase for /dev/nvme0n1p2:
cryptsetup open /dev/nvme0n1p2 root  7.22s user 0.12s system 135% cpu 5.432 total
root@archiso ~ # mount /dev/mapper/root /mnt
```

- Now create and mount the boot partition as normal. That one can't be encrypted

#### 6.2 Regular partition without encryption

- Format partitions using `mkfs` with this guide: https://man.archlinux.org/man/mkfs.8
- Root partition with ext4 filesystem using `mkfs.ext4`: https://wiki.archlinux.org/title/Ext4
```
root@archiso~# mkfs.ext4 /dev/sda3
```
- Create FAT 32 filesystem for UEFI boot partition with `mkfs.fat`: https://man.archlinux.org/man/mkfs.fat.8
```
root@archiso~# mkfs.fat -F 32 /dev/sda1
```
- Optional SWAP partition with `mkswap` and `swapon`: https://man.archlinux.org/man/mkswap.8
- SWAP wiki: https://wiki.archlinux.org/title/Swap
```
root@archiso~# mkswap /dev/sda2
root@archiso~# swapon /dev/sda2
```
- Mount the partitions using `mount`: https://wiki.archlinux.org/title/File_systems#Mount_a_file_system
- Optional `--mkdir` included in command if `/mnt/boot` isn't already created
```
root@archiso~# mount /dev/sda3 /mnt
root@archiso~# mount --mkdir /dev/sda1 /mnt/boot

root@archiso~# lsblk
NAME    MAJ:MIN RM  SIZE    RO  TYPE    MOUNTPOINTS
loop0   7:0     0   693.4M  1   loop    /run/archiso/airootfs
sda     8:0     0   100G    0   disk    
|-sda1  8:1     0   1G      0   part	/mnt/boot
|-sda2  8:2     0   8G      0   part	[SWAP]
|-sda3  8:3     0   91G     0   part	/mnt
sr0     11:0    1   813.3M  0   rom     /run/archiso/bootmnt
```
### 7. Section 2.1/2

(ignoring ranking mirrors for now)
- Use `pacstrap` to install linux onto the disk at `/mnt` alongside bare minimum packages needed
- https://man.archlinux.org/man/pacstrap.8
- The `-K` option: Initialize an empty pacman keyring in the target (implies -G).
```
root@archiso~#  pacstrap -K /mnt base linux linux-firmware vim networkmanager sudo openssh intel-ucode base-devel
```
- Also add amducode or intelucode

### 8. Section 3.1
- Now we need to generate a filesystem table for the boot sequence to find. This automatically mountds your disks upon startup.
- Use `genfstab` with the `-U` option to use UUID instead of disk labels. ***(THIS WILL BE IMPORTANT LATER)***
- `/mnt` is added to create the fstab for the `/mnt` disk
- `>>` appends lines to that file at the bottom.
```
root@archiso~# genfstab -U /mnt >> /mnt/etc/fstab
root@archiso~# cat /mnt/etc/fstab
# Static information about the filesystems.
# See fstab(5) for details

# <file system>					<dir>	<type>	<options>	<dump>	<pass>
# /dev/sda3
UUID=3533c3bf-a565-4a32-ae1b-404e40b856f6	/	ext4	rw,relatime	0	1

# /dev/sda1
UUID=C9DC-E53F					/boot	vfat	rw,relatime..	0	?

# /dev/sda2
UUID=a79ba5f3-0ca9-44d7-b72b-d7f15405acbc	none	swap	rw,relatime	0	0
```
### 9. Section 3.2/3/4/5
- Generate locale, keymap, hostname, hosts
```
root@archiso~# arch-chroot /mnt
[root@archiso /]# ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
[root@archiso /]# hwclock --systohc --utc
[root@archiso /]# vi /etc/locale.gen
```
{
	uncomment en_US.UTF-8 UTF-8
	save/exit
}
```
[root@archiso /]# locale-gen
Generating locales...
  en_US.UTF-8... done
Generation complete.
[root@archiso /]# echo "LANG=en_US.UTF-8" > /etc/locale.conf
[root@archiso /]# cat /etc/locale.conf
LANG=en_US.UTF-8
[root@archiso /]# echo "KEYMAP=us" > /etc/vconsole.conf
[root@archiso /]# cat /etc/vconsole.conf
KEYMAP=us
[root@archiso /]# echo "my-arch" > /etc/hostname
[root@archiso /]# cat /etc/hostname
my-arch
[root@archiso /]# vim /etc/hosts
[root@archiso /]# cat /etc/hosts
# Static table lookup for hostnames.
# See hosts(5) for details.
127.0.0.1       localhost
127.0.1.1       bobarch.localdomain     bobarch

::1     localhost       ip6-localhost   ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

### 10. Section 3.6/7 
- Enable file system trim on SSD using a timer service
```
[root@archiso /]# systemctl enable fstrim.timer
Created symlink /etc/systemd/system/timers.target.wants/fstrim.timer → /usr/lib/systemd/system/fstrim.timer.
```
- Enable 32 bit library support in `pacman.conf`
```
[root@archiso /]# vim /etc/pacman.conf
```
- Uncomment these two lines in `/etc/pacman.conf`:
```
[multilib]
Include /etc/pacman.d/mirrorlist
```
- Download multilib package data
```
[root@archiso /]# sudo pacman -Sy
:: Synchronizing package databases...
 core                                                       130.3 KiB   236 KiB/s 00:01 [###################################################] 100%
 extra                                                        8.3 MiB  12.1 MiB/s 00:01 [###################################################] 100%
 multilib                                                   143.2 KiB  1860 KiB/s 00:00 [###################################################] 100%
```

- Set root password
```
[root@archiso /]# passwd
```
- Add regular user:
```
[root@archiso /]# useradd -m -g users -G wheel,storage,power -s /bin/bash brandon
```
-m for creating home directory
-G for adding wheel,power,storage groups
-s for adding default bash at /bin/bash
-g users for adding user to the users group

- Set password for new user
```
[root@archiso /]# passwd brandon
*enter password
```
- Edit sudo users group
```
[root@archiso /]# visudo
```
- Uncomment first `%wheel` line
```
%wheel ALL=(ALL:ALL) ALL
```
- Add this line to `visudo` at the bottom to enforce using the root password for sudo
```
Defaults rootpw
```
- check efivars
```
[root@archiso /]# mount -t efivarfs efivarfs /sys/firmware/efi/efivars/
[root@archiso /]# ls /sys/firmware/efi/efivars/
```

### 10b
- Modify `/etc/mkinitcpio.conf` for encryption
```
[root@archiso /]# vim /etc/mkinitcpio.conf
```
- add `sd-encrypt` hook after udev or systemd hook.
```
HOOKS=(base udev autodetect modconf kms keyboard keymap consolefont block sd-encrypt filesystems fsck)
```
- Run `mkinitcpio` script with option `-P` for regenerating all the presets:
```
[root@archiso /]# mkinitcpio -P 
```


### 11. Section 3.8 (Installing systemd)
#### Reference: https://wiki.archlinux.org/title/Systemd-boot
```
[root@archiso /]# bootctl install
Created "/boot/EFI".
Created "/boot/EFI/systemd".
Created "/boot/EFI/BOOT".
Created "/boot/loader".
Created "/boot/loader/entries".
Created "/boot/EFI/linux".
Copied "/usr/lib/systemd/boot/efi/systemd-bootx64.efi" to "/boot/EFI/systemd/systemd-bootx64.efi".
Copied "/usr/lib/systemd/boot/efi/systemd-bootx64.efi" to "/boot/EFI/BOOT/BOOTX64.EFI".
Random seed file /boot/loader/random-seed successfully written (32 bytes).
Successfully initialized system token in EFI variable with 32 bytes.
Created EFI boot entry "Linux Boot Manager". 
```
- Now following: https://wiki.archlinux.org/title/Systemd-boot#Configuration
- (Checking for correct path to make sure)
```
[root@archiso /]# ls /boot
EFI initramfs-linux-linux-fallback.img initramfs-linux.img loader vmlinuz-linux
[root@archiso /]# ls /boot/loader
entries entries.srel loader.conf random-seed
```
```
[root@archiso /]# vi /boot/loader/loader.conf
```
- (what I see without making any changes:)
```
#timeout 3
#console-mode keep
```
- Create basic loader
```
[root@archiso /]# cat /boot/loader/loader.conf
default arch.conf
timeout 4
console-mode max
editor no
```
- Create loader entry
```
[root@archiso /]# vi /boot/loader/entries/arch.conf
[root@archiso /]# cat /boot/loader/entries/arch.conf
title Arch
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img
options root=PARTUUID=UUID=bf4342bd-52ca-470d-aca0-2164f829ea90 rw
```
- Install `intel-ucode` package
```
[root@archiso /]# pacman -S intel-ucode
[root@archiso /]# ls /boot
EFI  initramfs-linux-fallback.img  initramfs-linux.img  intel-ucode.img  loader  vmlinuz-linux
```
- Copy arch.conf to arch-fallback.conf and use linux-fallback.img
```
[root@archiso /]# vi /boot/loader/entries/arch-fallback.conf
[root@archiso /]# cat /boot/loader/entries/arch-fallback.conf
title Arch (fallback initramfs)
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux-fallback.img
options root=PARTUUID=UUID=bf4342bd-52ca-470d-aca0-2164f829ea90 rw

[root@archiso /]# bootctl list
         type: Boot Loader Specification Type #1 (.conf)
        title: Arch (default) (not reported/new)
           id: arch.conf
       source: /boot//loader/entries/arch.conf
        linux: /boot//vmlinuz-linux
       initrd: /boot//intel-ucode.img
               /boot//initramfs-linux.img
      options: root=PARTUUID=UUID=bf4342bd-52ca-470d-aca0-2164f829ea90 rw

         type: Boot Loader Specification Type #1 (.conf)
        title: Arch (fallback initramfs) (not reported/new)
           id: arch-fallback.conf
       source: /boot//loader/entries/arch-fallback.conf
        linux: /boot//vmlinuz-linux
       initrd: /boot//intel-ucode.img
               /boot//initramfs-linux-fallback.img
      options: root=PARTUUID=UUID=bf4342bd-52ca-470d-aca0-2164f829ea90 rw
```
- Trick to find UUID of root partition, but doesn't work with LUKS encryption
```
[root@archiso /]# echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/sda3) rw" >> /boot/loader/entries/arch.conf
[root@archiso /]# echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/sda3) rw" >> /boot/loader/entries/arch-fallback.conf
```

### 12.Creating hook to update systemd (section 1.3.2.2)
```
[root@archiso /]# mkdir /etc/pacman.d/hooks
[root@archiso /]# vi /etc/pacman.d/hooks/95-systemd-boot.hook
[root@archiso /]# cat /etc/pacman.d/hooks/95-systemd-boot.hook
[Trigger]
Type = Package
Operation = Upgrade
Target = systemd

[Action]
Description = Gracefully upgrading systemd-boot...
When = PostTransaction
Exec = /usr/bin/systemctl restart systemd-boot-update.service
```

```
[root@archiso /]# exit
```
```
root@archiso~# reboot
```

### 13. Installing GPU drivers
- check this guide out: https://github.com/lutris/docs/blob/master/InstallingDrivers.md
```

```


### References

https://man.archlinux.org/man/genfstab.8

hooks for updating systemd: https://archlinux.org/pacman/alpm-hooks.5.html

https://wiki.archlinux.org/title/Systemd-boot

https://man.archlinux.org/man/loader.conf.5

https://wiki.archlinux.org/title/GPT_fdisk





Enable Network
```
systemctl start NetworkManager.service
systemctl status NetworkManager.service
```
```
ping google.com
```
