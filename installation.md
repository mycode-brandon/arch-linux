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

## Important commands to know in Linux terminal:
- `mkdir`: creates a directory (a folder)
    - For example:
      ```
      root@archiso~# mkdir newfolder
      ```
- `ls`: ls stands for list. This lists the files in the current directory or one you specifiy.
    - For example listing the files inside the current directory (a folder)
      ```
      root@archiso~# mkdir newfolder
      root@archiso~# mkdir newfolder/subfolder
      root@archiso~# ls
      newfolder
      root@archiso~# ls newfolder
      subfolder
      ```

- `cat`: prints the contents of a file to the screen
- `echo` combined with `>` or `>>`: We can directly add text to a file using this combination.
- `>`: overwrites everything on the file to the right of the sign.
- `>>`: appends to the bottom of the file on the right of the sign.
  - For example printing to the screen:
    ```
    root@archiso~# echo "Hello"
    Hello
    ```
  - For example overwriting a file:
    ```
    root@archiso~# echo "Hello" > example.txt
    root@archiso~# cat example.txt
    Hello
    root@archiso~# echo "Hello Again" > example.txt
    root@archiso~# cat example.txt
    Hello Again
    ```
  - For example appending to a file:
    ```
    root@archiso~# echo "Hello" >> example.txt
    root@archiso~# cat example.txt
    Hello
    root@archiso~# echo "Hello Again" >> example.txt
    root@archiso~# cat example.txt
    Hello
    Hello Again
    ```

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
[iwd]# device list
[iwd]# device device set-property Powered on
[iwd]# adapter adapter set-property Powered on
[iwd]# station device scan
[iwd]# station device get-networks
[iwd]# station device connect SSID
```
- or use this command to connect:
```
iwctl --passphrase passphrase station device connect SSID
```

### 2. Check disks available with `lsblk`
```
root@archiso~# lsblk
NAME    MAJ:MIN RM  SIZE    RO  TYPE    MOUNTPOINTS
loop0   7:0     0   693.4M  1   loop    /run/archiso/airootfs
sda     8:0     0   100G    0   disk    
sr0     11:0    1   813.3M  0   rom     /run/archiso/bootmr
```
### 3. Wipe disk you want to install on to start fresh with `gdisk`
```
root@archiso~# gdisk /dev/sda
```
- Interactive Actions: {
expert mode (x), 
zap (z), 
yes
}

### 4. Section 1.5/6/7/8 of Installation Guide
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
root@archiso~# timedatectl set-ntp true
```

### 5. Section 1.9 of Installation Guide - Partition the disks

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
```
root@archiso~#  pacstrap -K /mnt base linux linux-firmware vim networkmanager sudo openssh
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
```

### 10. Section 3.6/7 
```
[root@archiso /]# mkinitcpio -P (No ERRORS)
[root@archiso /]# passwd
```
{
	enter password
}


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
[root@archiso /]# 
[root@archiso /]# 
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
- (creating a basic one based on example)
```
[root@archiso /]# cat /boot/loader/loader.conf
default arch.conf
timeout 4
console-mode max
editor no

[root@archiso /]# vi /boot/loader/entries/arch.conf
[root@archiso /]# cat /boot/loader/entries/arch.conf
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root="LABEL=Arch OS" rw
```
- (initrd  /intel-ucode.img  omitted due to being in VM)
```
[root@archiso /]# vi /boot/loader/entries/arch-fallback.conf
[root@archiso /]# cat /boot/loader/entries/arch-fallback.conf
title   Arch Linux (fallback initramfs)
linux   /vmlinuz-linux
initrd  /initramfs-linux-fallback.img
options root="LABEL=Arch OS" rw

[root@archiso /]# bootctl list
      type: Boot Loader Specification Type #1 (.conf)
     title: Arch Linux (default) (not reported/new)
        id: arch.conf
    source: /boot//loader/entries/arch.conf
     linux: /boot//vmlinuz-linux
    initrd: /boot//initramfs-linux.img
   options: /root="LABEL=Arch OS" rw

      type: Boot Loader Specification Type #1 (.conf)
     title: Arch Linux (fallback initramfs) (not reported/new)
        id: arch-fallback.conf
    source: /boot//loader/entries/arch-fallback.conf
     linux: /boot//vmlinuz-linux
    initrd: /boot//initramfs-linux-fallback.img
   options: /root="LABEL=Arch OS" rw
```
- ***NOTE: This will not boot. This is a common error and the WIKI doesn't do a good job here.***
- In the `arch.conf` and `arch-fallback.conf`, it uses `/root=LABEL....`, however we are using UUID's. This is misleading in the wiki.
- We need to add the UUID instead of the label. Here is a simple command that automatically adds the UUID to each file:
```
[root@archiso /]# echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/sda3) rw" >> /boot/loader/entries/arch.conf
[root@archiso /]# echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/sda3) rw" >> /boot/loader/entries/arch-fallback.conf
```
- **Make sure to delete the old line that includes the label**

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

```
[root@archiso /]# cat /boot/loader/entries/arch.conf
title Arch Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
options root=PARTUUID=23a1ac3a-12a1-4eab-9a8f-db0b7427d307 rw

[root@archiso /]# cat /boot/loader/entries/arch-fallback.conf
title Arch Linux (fallback initramfs)
linux /vmlinuz-linux
initrd /initramfs-linux-fallback.img
options root=PARTUUID=23a1ac3a-12a1-4eab-9a8f-db0b7427d307 rw
```
### References

https://man.archlinux.org/man/genfstab.8

hooks for updating systemd: https://archlinux.org/pacman/alpm-hooks.5.html

https://wiki.archlinux.org/title/Systemd-boot

https://man.archlinux.org/man/loader.conf.5

https://wiki.archlinux.org/title/GPT_fdisk



Add User:
```
useradd -m -G wheel -s /bin/bash brandon
```
-m for creating home directory
-G for adding wheel group
-s for adding default bash
```
passwd brandon
*enter password
```
```
visudo
```
{
	uncomment first %wheel command
}

Enable Network
```
systemctl start NetworkManager.service
systemctl status NetworkManager.service
```
```
ping google.com
```
