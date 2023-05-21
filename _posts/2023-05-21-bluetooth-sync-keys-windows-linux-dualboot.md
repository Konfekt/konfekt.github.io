---
layout: post
title: "Syncing Bluetooth Pairings between Linux and Microsoft Windows 10/11 on a dual boot computer"
date: 2023-05-21
categories: Sync Bluetooth Pairing Linux Microsoft Windows 10 11 dual boot
comments: true
---

To sync the Bluetooth pairing keys between Linux and Microsoft Windows 10/11 on a dual boot computer, so that every Bluetooth device works on Linux just as well as on Microsoft Windows, without having to (re)pair it on every reboot:

0. Pair the Bluetooth devices in Microsoft Windows.
0. Then, in Linux, pair the Low Energy devices last, but first all other ones:

# Non Low-Energy devices

- For non-Low-Energy devices, use [bt-dualboot](https://github.com/Simon128/bt-dualboot), that exports the pairing keys to Microsoft Windows from Linux as follows:

    0. Install `chntpw` by your package manager, say `sudo zypper install chntpw` on Opensuse.
    0. install `bt-dualboot` by `sudo pip install git+https://github.com/Simon128/bt-dualboot`
    0. Pair the Bluetooth devices on Linux.
    0. run `lsblk` to find out your Microsoft Windows partition, say `/dev/sdb1`, and mount it by `sudo mount /dev/sdb1 /mnt`.
    0. run it by `sudo bt-dualboot --backup --sync-all` to export the pairing keys from Linux's `/var/lib/bluetooth` folder into the Microsoft Windows registry.

# Low-Energy devices

- for newer Low Energy devices, such as Microsoft Designer Mouse and Keyboard, use [this script](https://gist.github.com/Mygod/f390aabf53cf1406fc71166a47236ebf/raw/8514b2bd949c1f56a8d922ac284345b489dee871/export-ble-infos.py) that imports the pairing keys from Microsoft Windows to Linux, as follows:

    0. Install `chntpw` by your package manager, say `sudo zypper install chntpw` on Opensuse.
    0. Download [the script](https://gist.github.com/Mygod/f390aabf53cf1406fc71166a47236ebf/raw/8514b2bd949c1f56a8d922ac284345b489dee871/export-ble-infos.py) as `export-ble-infos.py` into, say, `~/bin`.
    0. Run `lsblk` to find out your Microsoft Windows partition, say `/dev/sdb1`, and mount it by `sudo mount /dev/sdb1 /mnt`.
    0. run

        ```sh
        $ cd ~/bin
        $ chmod a+x ./export-ble-infos.py
        $ ./export-ble-infos.py
        $ sudo bash -c 'cp -r ./bluetooth /var/lib && systemctl restart bluetooth'
        $ rm -r bluetooth
        ```

    to import all Bluetooth Low Energy devices from the Microsoft Windows registry into Linux's `/var/lib/bluetooth` folder.

# Manually

If you need to export your Bluetooth keys from a Windows Virtual machine to Linux, then [manual effort](https://unix.stackexchange.com/questions/568521/simpler-method-of-pairing-bluetooth-devices-for-both-windows-linux) seems called for.
To export the Bluetooth key from the Windows registry, either [add your user to those having access permissions to its folder](https://superuser.com/questions/1677881/bluetooth-pairing-on-dual-boot-of-windows-11-linux-or-some-os-else/1682149#1682149) or download [psexec](http://live.sysinternals.com/psexec.exe) to start `regedit` by `psexec -s -i regedit.exe` for additional access rights.
