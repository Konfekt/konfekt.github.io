---
layout: post
title: "Restoring a Linux installation"
date: 2017-03-26
categories: fsarchiver grub fstab
comments: true
---

# Aim

Restore a Linux installation,

- the root `/` and
- the home `/home/$USER` directory

onto another hardware.

We assume that the installed Linux system (in our case OpenSUSE 42.2) uses

- the bootloader grub2

and has separate partitions for

- the root `/` and
- the home `/home` folders


As a backup tool, we will use `fsarchiver`.
Its advantages:

- fast
- stable, and
- free of dependencies, that is, a single executable.

# Saving the installation

## Saving the partitions

We assume that the backup partition is of type `NTFS`, the file system used by Microsoft Windows (32 bit).
Otherwise, use `lsblk` to recognize the device letter `L` and number `N`, and mount your partition by `mount /dev/sdLN $MOUNT`.

Configure our backup so that it

- mounts inside `/mnt` our backup medium (of NTFS file-system)
- labeled `USB_BKP`, and
- saves the images to the subfolder `backup` (of the mounted folder):

```sh
MOUNT=/mnt
BKP_LABEL=USB_BKP
BKP_SUBFOLDER=backup

BKP_MOUNT="$MOUNT"/"$BKP_LABEL"
mkdir --parents "$BKP_MOUNT"
mount --types ntfs LABEL="$BKP_LABEL" --target "$BKP_MOUNT"
```

Save images of the root and home partition, skipping folders containing temporary and cache files:

```sh
BKP_FOLDER="$BKP_MOUNT"/"$BKP_SUBFOLDER"

HOME_FOLDER="/home/$USER"
fsarchiver --allow-rw-mounted \
  --exclude="$HOME_FOLDER/.local/share/Trash" \
  --exclude="$HOME_FOLDER/.cache" \
  savedir "$BKP_FOLDER/home.fsa" "$HOME_FOLDER"
# ROOT_FOLDER="/"
fsarchiver --allow-rw-mounted \
  --exclude=/home \
  --exclude='/media' \
  --exclude='/mnt' \
  --exclude='/run' \
  --exclude='/dev' \
  --exclude='/proc' \
  --exclude='/sys' \
  --exclude='/tmp' \
  --exclude='/var/run' \
  --exclude='/var/lock' \
  --exclude='/lib/modules/*/volatile/.mounted' \
  --exclude='/.snapshots' \
  savedir "$BKP_FOLDER/root.fsa" "$ROOT_FOLDER"

umount --verbose "$BKP_MOUNT"
rmdir --verbose "$BKP_MOUNT"
```

## Save the backup program

This is not strictly necessary, but avoids depending on fsarchiver been installed on the rescue system or not:
Copy the `fsarchiver` executable, say `/usr/sbin/fsarchiver` into the `$BKP_FOLDER` by

```sh
cp /usr/sbin/fsarchiver $BKP_FOLDER
```

## Save the file-system table

This is not strictly necessary, but avoids editing manually certain IDs in the file-system table file `/etc/fstab` after restoring the root partition:
Copy it into the `$BKP_FOLDER` by
```sh
mkdir -p $BKP_FOLDER/etc
cp /etc/fstab $BKP_FOLDER/etc/fstab
```

To explain:
The file `/etc/fstab` mounts file systems, partitions, to directories, for example the root partition `/dev/sda1` to `/` and the home partition `/dev/sda2` to `/home`.
These file systems are identified by a hexadecimal string, the universal unique identifier (UUID).

When reinstalling, these UUID are reassigned.
The bootloader uses these reassigned UUIDs,
We therefore save `/etc/fstab` and restore it afterwards, and thus keep the UUIDs as known to the bootloader.

# Restoring the installation

## Reinstalling Linux

Install the Linux distribution once in its most basic configuration.
This serves us to

- partition the hard drive, and
- install the bootloader (Grub2 in our case).

Once Linux is installed, boot into a rescue system, such as that on the installation DVD (or USB Stick) of the Linux distribution.
Log into a terminal.
We use `ls -l /dev/disk/by-uuid` to detect which device letter (say `/dev/sda`) is assigned to which hard disk and which number to which partition (say `/dev/sda1`).
We note letter and number of the

- backup partition, as well as
- the root and home partitions.

## Restoring the partitions

We assume that

- the backup partition is `/dev/sdb1`, and
- the root and home partitions are `/dev/sda1` and `/dev/sda2`.

We mount and then restore the files:

```sh
mkdir -p /mnt/bkp
mkdir -p /mnt/root
mkdir -p /mnt/home
mount /dev/sdb1 /mnt/bkp
mount /dev/sda1 /mnt/root
mount /dev/sda2 /mnt/home

rm -rf /mnt/root
rm -rf /mnt/home
chroot /mnt/bkp/
  ./fsarchiver restdir root.fsa /mnt/root;
  ./fsarchiver restdir home.fsa /mnt/home
exit
```

We also add empty directories of all those skipped when backing them up:

```sh
chroot /mnt/root
  mkdir -p /home
  mkdir -p /media
  mkdir -p /mnt
  mkdir -p /run
  mkdir -p /dev
  mkdir -p /proc
  mkdir -p /sys
  mkdir -p /tmp
  mkdir -p /var/run
  mkdir -p /var/lock
exit
```

## Restoring the bootloader

We assume (as above) that the root partition is `/dev/sda1`.

Then

```sh
mkdir -p /mnt/root
mount /dev/sda1 /mnt/root
mkdir -p /mnt/root/dev
mount --bind /dev /mnt/root/dev

chroot /mnt/root
  mkdir -p /mnt/root/proc
  mount -t proc proc /proc
  mkdir -p /mnt/root/sys
  mount -t sysfs sysfs /sys

  grub2-mkconfig -o /boot/grub2/grub.cfg
  grub2-install /dev/sda
exit

```

## Restoring /etc/fstab

Either

- you saved `/etc/fstab` before restoring the partitions, then copy it to `/etc/fstab` by

```sh
mkdir -p /mnt/bkp;
mkdir -p /mnt/root
mount /dev/sdb1 /mnt/bkp;
mount /dev/sda1 /mnt/root
cp /mnt/bkp/etc/fstab /mnt/root/etc/fstab
```
or

- otherwise, use `ls -l /dev/disk/by-uuid` to update the UUID entries in `/etc/fstab` accordingly, say by `vim /etc/fstab`.

