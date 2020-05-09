---
layout: post
title: "Restoring a Linux installation"
date: 2017-03-26
categories: fsarchiver grub fstab
comments: true
---

We want to restore a Linux installation, that is, its

- root `/` and
- home `/home/$USER` directory

onto another computer.

# Premises

We assume that the installed Linux system (in our case, it happened to be OpenSUSE 42.2) uses

- the bootloader `grub2`

and has separate partitions for

- the root `/` and
- the home `/home` folders

As a backup tool, we will use `rsync`, because it is

- stable,
- fast
- versatile, and
- free of dependencies, that is, it is a single executable.

# Saving the Installation

First, while not strictly necessary, copying the `rsync` executable, say `/usr/bin/rsync` into the `$BKP_FOLDER` by

```sh
cp /usr/bin/rsync $BKP_FOLDER
```

avoids depending on whether `rsync` has been installed on the rescue system or not.

To save the partitions:

<!-- We assume that the backup partition is of type `NTFS`, the file system used by Microsoft Windows (32 bit). -->

1. Use `lsblk` to identify the backup partition: its device letter, say `b`, and its partition number, say `1`; for example `/dev/sdb1`.
1. Mount the partition `$PARTITION` to a folder `$FOLDER,` say `/mnt`, by `mount $PARTIION $FOLDER`, say `mount /dev/sdb1 /mnt`:
    <!-- To save our data, we choose `backup` as name of the folder in our backup partition: -->

    ```sh
    MOUNT=/mnt
    BKP_LABEL=USB_BKP

    BKP_MOUNT="$MOUNT"/"$BKP_LABEL"
    mkdir --parents "$BKP_MOUNT"
    mount /dev/sdb1 "$BKP_MOUNT"
    ```

1. Copy

    - the home partition, skipping trash and cache files, and
    - the root partition, skipping the home folder (which is on a separate partition) and folders that only contain

        - boot images and kernel modules,
        - backups
        - removable media, and
        - temporary and cache files:

    ```sh
    BKP_FOLDER="$BKP_MOUNT"

    HOME_FOLDER="/home/$USER"
    HOME_BACKUP_FOLDER="$BKP_FOLDER/home"
    mkdir --parents "$HOME_BACKUP_FOLDER"

    rsync -avxEHA --delete --human-readable --info=progress2 \
      --exclude="/.local/share/Trash" \
      --exclude="/.cache" \
      "$HOME_FOLDER/" "$HOME_BACKUP_FOLDER/"

    ROOT_FOLDER="/"
    ROOT_BACKUP_FOLDER="$BKP_FOLDER/root"
    mkdir --parents "$BKP_FOLDER/root"

    rsync -avxEHA --delete --human-readable --info=progress2 \
      --exclude=/home \
      --exclude='/etc/fstab' \
      --exclude='/boot/' \
      --exclude='/lib/' \
      --exclude='/lib64/' \
      --exclude='/.snapshots' \
      --exclude='/media' \
      --exclude='/mnt' \
      --exclude='/run' \
      --exclude='/dev' \
      --exclude='/proc' \
      --exclude='/sys' \
      --exclude='/tmp' \
      --exclude='/var/run' \
      --exclude='/var/lock' \
      --exclude='/var/tmp' \
      "$ROOT_FOLDER" "$ROOT_BACKUP_FOLDER/"

    umount --verbose "$BKP_MOUNT"
    rmdir --verbose "$BKP_MOUNT"
    ```

<!-- Saving the file-system table is not strictly necessary, but avoids editing manually certain IDs in the file-system table file `/etc/fstab` after restoring the root partition:  -->
<!-- Copy it into the `$BKP_FOLDER` by  -->
<!-- ```sh  -->
<!-- mkdir -p $BKP_FOLDER/etc  -->
<!-- cp /etc/fstab $BKP_FOLDER/etc/fstab  -->
<!-- ```  -->
<!--   -->
<!-- To explain:  -->
<!-- The file `/etc/fstab` mounts file systems, partitions, to directories, for example the root partition `/dev/sda1` to `/` and the home partition `/dev/sda2` to `/home`.  -->
<!-- These file systems are identified by a hexadecimal string, the universal unique identifier (UUID).  -->
<!--   -->
<!-- When reinstalling, these UUID are reassigned.  -->
<!-- The bootloader uses these reassigned UUIDs,  -->
<!-- We therefore save `/etc/fstab` and restore it afterwards, and thus keep the UUIDs as known to the bootloader.  -->

# Restoring the installation

To restore the installation,

1. reinstall Linux minimally,
1. copy the partitions, and
1. finally restore the bootloader:

## Reinstalling Linux

1. To

    - partition the hard drive, and
    - install the bootloader (Grub2 in our case),

    install the Linux distribution in its most basic configuration (that is, installing as few packages as possible).

2. Once Linux is installed, boot into a rescue system, such as that on the installation medium (DVD or USB drive) of the Linux distribution.
1. Log into a terminal.
1. Use `ls -l /dev/disk/by-uuid` to detect which device letter (say `/dev/sda`) is assigned to which hard disk and which number to which partition.
1. Note the letter and number of the

    - backup partition, as well as
    - the root and home partitions.

    (For example, `/dev/sda` might be the hard disk, `\dev\sda2` the root partition and `\dev\sda3` the home partition).

## Restoring the partitions

We assume that

- the backup partition is `/dev/sdb1`, and
- the root and home partitions are `/dev/sda1` and `/dev/sda2`.

To first mount and then restore the files:

```sh
mkdir -p /mnt/bkp
mkdir -p /mnt/root
mkdir -p /mnt/home

mount /dev/sdb1 /mnt/bkp
mount /dev/sda1 /mnt/root
mount /dev/sda2 /mnt/home

cd /mnt/bkp/
rsync -avxEHA --delete --human-readable --info=progress2 \
root/ /mnt/root/;
rsync -avxEHA --delete --human-readable --info=progress2 \
home/ /mnt/home/
```

## Restoring the bootloader

We assume (as above) that the root partition is `/dev/sda1`.
To restore the bootloader:

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

<!-- ## Restoring /etc/fstab -->
<!--  -->
<!-- Either -->
<!--  -->
<!-- - you saved `/etc/fstab` before restoring the partitions, then copy it to `/etc/fstab` by -->
<!--  -->
<!-- ```sh -->
<!-- mkdir -p /mnt/bkp; -->
<!-- mkdir -p /mnt/root -->
<!-- mount /dev/sdb1 /mnt/bkp; -->
<!-- mount /dev/sda1 /mnt/root -->
<!-- cp /mnt/bkp/etc/fstab /mnt/root/etc/fstab -->
<!-- ``` -->
<!-- or -->
<!--  -->
<!-- - otherwise, use `ls -l /dev/disk/by-uuid` to update the UUID entries in `/etc/fstab` accordingly, say by `vim /etc/fstab`. -->
