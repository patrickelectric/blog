---
title: "Raspberry Pi with multiboot (multiple rootfs)"
header:
    teaser: assets/raspberry_multiboot/gparted.png
tags:
    - raspberry
    - boot
---

One question that I receive (from my friends, myself and people that install archlinux at 3am) appears every time. _How to create a multiboot architecture inside a sdcard for the Raspberry Pi ?_

After a boring day doing some unrelated stuff out of home, and with 1h free to do something before sleep and eating (heating my frozen lasagna), why not solving that ~~secular~~ question that follow me ?

I took some time to think about it: [maybe doing something related to the archlinux boot section](https://wiki.archlinux.org/index.php/Category:Boot_loaders), or with [noobs](https://github.com/raspberrypi/noobs), and maybe with [BerryBoot](https://www.berryterminal.com/doku.php/berryboot).

With 1h-ish free I did remove all non-trivial ways.
 - archlinux: The script will use all the sdcard, changing the scripts will take time.
 - noobs: It does not appears to be the correct tool, some investigation shows that.
 - BerryBoot: Appears to be the way to go, but it'll take time also to configure everything.

# Doing the hard stuff with no sweat

So, the idea now is to break everything and try something new. **Use Raspbian image, create a second rootfs partition, change the rootfs in /boot**, sounds simple, easy and fast !

1. [Download Raspbian](https://www.raspberrypi.org/downloads/raspbian/) (I did everything with Raspbian stretch lite).
2. Burn the sdcard with Etcher (I like dd, but Etcher is so simple!).
3. Use gparted to create a second partition with the same configuration of the /rootfs one ! (rootfs2).

    ![LaKademy2017](/assets/raspberry_multiboot/gparted.png)

4. Make a copy of our old *rootfs*. `sudo cp /mount_point_of_rootfs/* /mount_point_of_rootfs2/* -r`
5. Now, with everything ready, we should update the partuuid values of `/etc/fstab` and `/boot/cmdline.txt`. (`sudo blkid`)
    ```
    /dev/mmcblk0p1: LABEL="boot" UUID="3725-1C05" TYPE="vfat" PARTUUID="0155ef5b-01"
    /dev/mmcblk0p2: LABEL="rootfs" UUID="fd695ef5-f047-44bd-b159-2a78c53af20a" TYPE="ext4" PARTUUID="0155ef5b-02"
    /dev/mmcblk0p3: LABEL="rootfs2" UUID="3da53eaa-4044-4554-8052-0f8c225f7b8c" TYPE="ext4" PARTUUID="0155ef5b-03"
    ```
6. With all partitions mounted, update:
    - `/boot/cmdline.txt`: partuuid of `/rootfs` to 0155ef5b-**02**
    - `/rootfs/etc/fstab`: partuuid of `/boot` to 0155ef5b-**01** and `/` to 0155ef5b-**02**
    - `/rootfs2/etc/fstab`: partuuid of `/boot` to 0155ef5b-**01** and `/` to 0155ef5b-**03**
7. To change which rootfs you want to boot, just need to change the suffix of the partuuid in `/boot/cmdline.txt`.

The **01** suffix is the boot partition (`/boot`), **02** is our "recovery" partition with 700MB free (`/rootfs`), and our last and probably the "main" partition with 13GB free is the one with **03** suffix (`/rootfs2`).

## Notes

- This may not work with different kernels because of the initramfs.
- If you want to improve this article, please send a [PR in my github](http://github.com/patrickelectric/patrickelectric.github.io) !
