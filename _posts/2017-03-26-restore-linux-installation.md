---
layout: post
title: "Restoring a Linux installation"
date: 2017-03-26
categories: fsarchiver grub fstab
comments: true
---

We want to copy a Linux installation, that is, its

- root `/` and
- home `/home/$USER` directory

onto another computer.

# Premises

We assume that the installed Linux system  uses

- the bootloader `grub2`

and has separate partitions for

- the root `/` and
- the home `/home` folders

# 1. Identify Devices
```bash
lsblk -f
```
- Source root partition: e.g. `/dev/sdaX`
- Target USB partition: e.g. `/dev/sdb1`

# 2. Prepare USB
```bash
mkfs.ext4 /dev/sdb1
mount /dev/sdb1 /mnt/usb
```

If USB also has EFI partition, then
```bash
mount /dev/sdb2 /mnt/usb/boot/efi
```

If not yet, then create a separate EFI partition if system boots via UEFI:
```bash
mkfs.vfat -F32 /dev/sdb2
mkdir -p /mnt/usb/boot/efi
mount /dev/sdb2 /mnt/usb/boot/efi
```

# 3. Backup Configs to Preserve  

```bash  
cp -a /mnt/usb/etc/fstab /mnt/usb/etc/fstab.backup
cp -a /mnt/usb/etc/passwd /mnt/usb/etc/passwd.backup
cp -a /mnt/usb/etc/group  /mnt/usb/etc/group.backup
cp -a /mnt/usb/etc/subgid /mnt/usb/etc/subgid.backup
cp -a /mnt/usb/etc/subuid /mnt/usb/etc/subuid.backup
cp -a /mnt/usb/etc/shadow /mnt/usb/etc/shadow.backup
cp -a /mnt/usb/etc/gshadow /mnt/usb/etc/gshadow.backup
```

# 3. Clone Root with `rsync`
# 3. Rsync Root Excluding Critical Paths  
```bash  
rsync -aAXHv --numeric-ids --delete --compress --info=stats1,progress2 --human-readable \
  --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} \
  --exclude={"/home/*","/etc/fstab","/etc/passwd","/etc/group","/etc/shadow"} \
  / /mnt/usb
```  
- `-a` archive (preserve perms, timestamps, symlinks)
- `-A` preserve ACLs
- `-X` preserve xattrs
- `--numeric-ids` preserve UID/GID mapping
- Exclude `/home` to keep existing partition intact.  
- Exclude `fstab` and user database to retain USB-specific data.  

# 4. Restore Preserved Files  
```bash
cp -a /mnt/usb/etc/fstab.backup /mnt/usb/etc/fstab
cp -a /mnt/usb/etc/passwd.backup /mnt/usb/etc/passwd
cp -a /mnt/usb/etc/group.backup  /mnt/usb/etc/group
cp -a /mnt/usb/etc/shadow.backup /mnt/usb/etc/shadow
```  
  
# 5. Verify `/etc/fstab`  
Ensure correct filesystem UUIDs for:  
- root filesystem (USB root partition)  
- home partition (USB `/home`)  
- EFI (if UEFI system)  
  
Check UUIDs: `blkid`.

# 6. Bind Mount for `chroot`
If the source file system is not BTRFS, say Ext4 or XFS, then

```bash
mount --rbind /dev /mnt/usb/dev
mount --rbind /proc /mnt/usb/proc
mount --rbind /sys /mnt/usb/sys
mount --rbind /run /mnt/usb/run
chroot /mnt/usb /bin/bash
```

Otherwise, that is, if the source file system uses BTRFS, then

```bash
mount /dev/sdb1 -o subvol=/@/boot/grub2/x86_64-efi  /mnt/usb/boot/grub2/x86_64-efi;
mount /dev/sdb1 -o subvol=/@/boot/grub2/i386-pc     /mnt/usb/boot/grub2/i386-pc;
mount /dev/sdb1 -o subvol=/@/var                    /mnt/usb/var;
mount /dev/sdb1 -o subvol=/@/usr/local              /mnt/usb/usr/local;
mount /dev/sdb1 -o subvol=/@/tmp                    /mnt/usb/tmp;
mount /dev/sdb1 -o subvol=/@/srv                    /mnt/usb/srv;
mount /dev/sdb1 -o subvol=/@/root                   /mnt/usb/root;
mount /dev/sdb1 -o subvol=/@/opt                    /mnt/usb/opt;
```

# 7. Inside Chroot: Recreate GRUB

## For BIOS:
```bash
grub-install /dev/sdb
update-grub
```
On Opensuse: 
```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
grub2-install /dev/sdb
```

## For UEFI:
```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=USB --recheck
update-grub
```
On Opensuse: 
```bash
sudo grub2-mkconfig -o /boot/efi/EFI/opensuse/grub.cfg
grub2-install /dev/sdb
```

# 8. Exit and Cleanup
```bash
exit
umount -R /mnt/usb
```

