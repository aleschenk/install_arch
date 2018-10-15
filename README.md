# INSTALL ARCH

## Setup keyboard
```console
root@archiso ~ # loadkeys [es|la-latin1]
```

## Connect wifi
```console
root@archiso ~ # wifi-menu
```

## Update the system clock
```console
root@archiso ~ # timedatectl set-ntp true
```

## Set Time Zone
```console
root@archiso ~ # timedatectl set-timezone America/Argentina/Buenos_Aires
```

## Verify the boot mode
```console
root@archiso ~ # ls /sys/firmware/efi/efivars
```
If the directory does not exist, the system may be booted in BIOS or CSM mode.

## Backup MBR/GPT
### Option 1:
```console
root@archiso ~ # sfdisk -d /dev/sda > sda.dump
```

### Option 2:
```console
root@archiso ~ # sgdisk -b=sgdisk-sda.bak
```

## CREATE GPT Table
```console
root@archiso ~ # parted /dev/sda mklabel gpt
```

## PARTITIONS
|                |        MBR       |        GPT       |
|---------------:|-----------------:|-----------------:|
|   Dialog       |     fdisk        |      fdisk       |
|                |     parted       |      gdisk       |
|                |                  |      parted      |
|                |                  |                  |
|Pseudo-graphics |     cfdisk       |      cfdisk      |
|                |                  |      cgdisk      |
|                |                  |                  |
|Non-interactive |      sfdisk      |      sfdisk      |
|                |      parted      |      sgdisk      |
|                |                  |      parted      |
|                |                  |                  |
|   Graphical    |     GParted      |      GParted     |
|				         | partitionmanager | partitionmanager |

```console
root@archiso ~ # sgdisk -n 1:2048:500M /dev/sda
root@archiso ~ # sgdisk -n 2:0:+1G /dev/sda
root@archiso ~ # sgdisk -n 3:0:0 /dev/sda
```

### WARNING the next commands will destroy all the data

(A fresh start)
Zero out all GPT and MBR data structures. Erase the GPT data structures so that the disk can be used as an MBR disk without concern that it might contain stray GPT data.
```console
root@archiso ~ # sgdisk -z /dev/sda
```

### The last parition is only needed on a GRUB, BIOS/GPT partitioning scheme.

### one line
```console
root@archiso ~ # sgdisk --new 1:2048:500M --type 1:8300 --change-name 1:"Linux /boot" \
                        --new 2:0:+6G --type 2:8300 -c 2:"/" \
                        --new 3:0:+1G --type 3:8300 --change-name 3:"Linux Swap" \
                        --new 4:0:+20M --type 4:ef02 --change-name 4:"Bios boot" /dev/sda
```

### same as above just in multiple lines
```console
root@archiso ~ # sgdisk --new 1:2048:500M --type 1:8300 --change-name 1:"Linux /boot" /dev/sda
root@archiso ~ # sgdisk --new 2:0:+6G     --type 2:8300 --change-name 2:"Linux /" /dev/sda
root@archiso ~ # sgdisk --new 3:0:+1G     --type 3:8200 --change-name 3:"Linux Swap" /dev/sda
root@archiso ~ # sgdisk --new 4:0:+20M    --type 4:ef02 --change-name 4:"Bios boot" /dev/sda
```



## ENCRYPTION

### Preparing non-boot partitions
```console
root@archiso ~ # cryptsetup -y -v luksFormat /dev/sdaX
```

### Default: 
```console
cryptsetup -v --cipher aes-xts-plain64 --key-size 256 --hash sha256 --iter-time 2000 --use-urandom --verify-passphrase luksFormat device
```

#### Unlocking/Mapping LUKS partitions with the device mapper
#### The name could be prefix_label ex: crypt_root, crypt_home, crypt_var etc.
#### cryptsetup open --type luks device [name]
```console
cryptsetup open --type luks /dev/sda1 crypt_root
```

## The name could be prefix_label ex: crypt_root, crypt_home, crypt_var etc.
```console
root@archiso ~ # cryptsetup open --type luks device [name]
```

## Default
```console
root@archiso ~ # cryptsetup -v --cipher aes-xts-plain64 --key-size 256 --hash sha256 --iter-time 2000 --use-urandom --verify-passphrase luksFormat device
```

## Unlocking/Mapping LUKS partitions with the device mapper
```console
root@archiso ~ # cryptsetup open --type luks /dev/sda1 crypt_root
```

## FORMAT with BTRFS
```console
root@archiso ~ # mkfs.btrfs -L linuxroot /dev/mapper/cryptroot
```

## FORMAT with EXT4
```console
root@archiso ~ # mkfs.ext4 -L linuxroot /dev/mapper/cryptroot
```

## SUBVOLUMENS (Onlfy for BTRFS)
```console
root@archiso ~ # mount -t btrfs -o compress=lzo /dev/mapper/cryptroot /mnt
root@archiso ~ # btrfs subvolume create /mnt/@
root@archiso ~ # btrfs subvolume create /mnt/@home
root@archiso ~ # btrfs subvolume create /mnt/@snapshots
```

```console
root@archiso ~ # umount /mnt
root@archiso ~ # mount -o compress=lzo,subvol=@ /dev/mapper/cryptroot /mnt
root@archiso ~ # mkdir -p /mnt/home
```

```console
root@archiso ~ # mount -o compress=lzo,subvol=@home /dev/mapper/cryptroot /mnt/home
root@archiso ~ # mkdir -p /mnt/.snapshots
```

```console
root@archiso ~ # mount -o compress=lzo,subvol=@snapshots /dev/mapper/cryptroot /mnt/.snapshots
root@archiso ~ # mkdir -p /var
```

```console
root@archiso ~ # btrfs subvolume create /mnt/var/tmp
root@archiso ~ # btrfs subvolume create /mnt/tmp
```

## MOUNTING EFI PARITION
```console
root@archiso ~ # mkdir -p /mnt/boot
root@archiso ~ # mount /dev/sda1 /mnt/boot
```

## INSTALLING BASE PACKAGE
```console
root@archiso ~ # pacstrap /mnt base btrfs-progs
```

## CONFIGURATION
```console
root@archiso ~ # genfstab -p /mnt >> /mnt/etc/fstab
root@archiso ~ # arch-chroot /mnt
```

## LOCALE

```
Uncomment: en_US.UTF-8 UTF-8 in /etc/locale.gen
Uncomment: en_US ISO-8859-1 in /etc/locale.gen
Uncomment: es_AR.UTF-8 UTF-8 in /etc/locale.gen
Uncomment: es_AR ISO-8859-1 in /etc/locale.gen
```

run: locale-gen

Create the file "/etc/locale.conf" with this content:

```bash
LANG=en_US.UTF-8
LANGUAGE=en_US
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC=es_AR.UTF-8
LC_TIME=es_AR.UTF-8
LC_COLLATE="en_US.UTF-8"
LC_MONETARY=es_AR.UTF-8
LC_MESSAGES="en_US.UTF-8"
LC_PAPER=es_AR.UTF-8
LC_NAME=es_AR.UTF-8
LC_ADDRESS=es_AR.UTF-8
LC_TELEPHONE=es_AR.UTF-8
LC_MEASUREMENT=es_AR.UTF-8
LC_IDENTIFICATION=es_AR.UTF-8
```

## KEYBOARD

Create the file "/etc/vconsole.conf" with this content:
KEYMAP=la-latin1

## TIMEZONE
```console
root@archiso ~ # ln -s /usr/share/zoneinfo/America/Argentina/Buenos_Aires /etc/localtime
```

## MK_INIT_CPIO
Add the encrypt hook to etc/mkinitcpio.conf.
It has to be before filesystems hook.

HOOKS="... encrypt ... filesystems ..."

## BOOTLOADER / GRUB
```console
root@archiso ~ # pacman -S grub efibootmgr
```
