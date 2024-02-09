# Installing Arch Linux
Reference: https://wiki.archlinux.org/title/Installation_guide

## Process:

### Download ISO, Flash USB
- Downlaod the ISO using a straight download or torrent method: https://archlinux.org/download/
- Copy the SHA256 key and paste into Notepad (if on Windows) or Linux text editor
- On Windows, use PowerShell to confirm the signature and that your ISO is complete and hasn't been tampered with.
  - `Get-FileHash .\archlinux-2023.08.01-x86_64.iso -Algorithm SHA256`
- On Linux
  - `sha256sum archlinux-2023.08.01-x86_64.iso`

### Connect to Internet

- Check internet connectivity: `ping google.com`
- If on WiFi, use `iwctl` to connect: https://wiki.archlinux.org/title/Iwd#iwctl
  - `root@archiso~# iwctl` to enter `iwd` command mode
  - `[iwd]# device list` to show devices
  - `[iwd]# device <device> set-property Powered on`
  - `[iwd]# adapter <adapter> set-property Powered on`
  - `[iwd]# station <device> scan`
  - `[iwd]# station <device> get-networks`
  - `[iwd]# station <device> connect <SSID>` *note: use quotes for SSID if there are spaces or single quotes, e.g. "My Network SSID"
  - `[iwd]# exit`
  - `root@archiso~# ping google.com` to verify internet connectivity

### Set Time, Keyboard, and Font

- `root@archiso~# loadkeys us`
- `root@archiso~# setfont ter-d32n`
- `root@archiso~# timedatectl set-timezone America/Chicago`
- `root@archiso~# timedatectl set-ntp true`

### Disk Partition

- Determine existing disk setup:
  - `root@archiso~# lsblk`
    
For example:
```
root@archiso~# lsblk
NAME    MAJ:MIN RM  SIZE    RO  TYPE    MOUNTPOINTS
loop0   7:0     0   693.4M  1   loop    /run/archiso/airootfs
sda     8:0     0   100G    0   disk    
sr0     11:0    1   813.3M  0   rom     /run/archiso/bootmr
```
- Wipe and prepare the chosen disk with `gdisk`. In the example above, it would be `sda`
  - `root@archiso~# gdisk /dev/sda`
  - `Command (? for help): x` - choose "x" for expert mode
  - `Expert command (? for help): z` - choose "z" to zap and clear entire drive
  - `About to wipe out GPT on /dev/nvme0n1. Proceed? (Y/N): y` - choose "y" for yes
  - `Blank out MBR? (Y/N): y` - choose "y" for yes
- Partition the disk using `cgdisk`.
  - `root@archiso~# cgdisk /dev/sda`
- Setup Boot Partition:
  - Press enter
  - Select `free space` with up/down arrow keys, and `New` with left/right arrow keys
  - `First Sector (...): <Enter>` - Press Enter to select default
  - `Size in sectors or...: 1G` - enter "1G" for 1 gigabyte. Overkill but fine.
  - `Hex code or GUID(L to show codes, Enter = 8300): EF00` - Enter "EF00" for Boot partition
  - `Enter new partition name...: boot` - Type "boot" to name boot partition
- Setup SWAP Partition:
  - Select `free space` with up/down arrow keys, and `New` with left/right arrow keys
  - `First Sector (...): <Enter>` - Press Enter to select default
  - `Size in sectors or...: 8G` - enter "8G" for 8 gigabytes. Whatever size you want.
  - `Hex code or GUID(...): 8200` - Enter "8200" for SWAP partition
  - `Enter new partition name...: swap` - Type "swap" to name swap partition
- Setup root Partition:
  - Select `free space` with up/down arrow keys, and `New` with left/right arrow keys
  - `First Sector (...): <Enter>` - Press Enter to select default
  - `Size in sectors or...: <Enter>` - enter "<Enter>" to use the rest of the disk. Can choose to create a partition for `/home` if wanted.
  - `Hex code or GUID(...): 8300` - Enter "8300" for Root partition
  - `Enter new partition name...: swap` - Type "root" to name root partition
- Write and Quit
  - Select `Write` using left/right arrow keys, press <Enter>
  - Select `Quit` using left/right arrow keys, press <Enter>

### Disk Formatting and Mounting Directories

- Make filesystem for root partition
  - `root@archiso~# mkfs.ext4 /dev/sda3`
- Make filesystem for boot partition
  - `root@archiso~# mkfs.fat -F 32 /dev/sda1`
- Make SWAP filesystem and turn SWAP on
  - `root@archiso~# mkswap /dev/sda2`
  - `root@archiso~# swapon /dev/sda2`

  - Mount the root and boot directories:
    - `root@archiso~# mount /dev/sda3 /mnt`
    - `root@archiso~# mount --mkdir /dev/sda1 /mnt/boot`
    - **NOTE: this order must be maintained. Mount /mnt before creating /mnt/boot, otherwise `genfstab` will have issues.**

Double check filesystem with `lsblk -o name,size,type,mountpoint,partlabel`
```
root@archiso~# lsblk -o name,size,type,mountpoint,partlabel
NAME    SIZE    TYPE  MOUNTPOINTS            PARTLABEL
loop0   693.4M  loop  /run/archiso/airootfs  
sda     100G    disk    
|-sda1  1G      part	/mnt/boot              boot
|-sda2  8G      part	[SWAP]                 swap
|-sda3  91G     part	/mnt                   root
sr0     813.3M  rom   /run/archiso/bootmnt
```

### Select Mirrors and Pacstrap the system

- You can opt to skip selecting mirrors

- Pacstrap the new install:
  - `root@archiso~#  pacstrap -K /mnt base linux linux-firmware neovim networkmanager sudo openssh intel-ucode base-devel man-db man-pages`
  - Choose whatever *-ucdoe you have, AMD or Intel.
  - Ensure you have a network package with dns, wifi, and whatever capabilties you need. Network manager should take care of all that.
  - **NOTE: the `-K` option for `pacstrap` is important, packages may not download properly without that.**
  - Don't forget the target directory: `/mnt`
