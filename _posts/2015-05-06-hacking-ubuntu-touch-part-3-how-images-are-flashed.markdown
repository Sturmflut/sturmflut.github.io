---
layout: post
title:  "Hacking Ubuntu Touch, Part 3: How images are flashed"
date:   2015-05-06 20:30:00
categories: Ubuntu Touch
---

*NOTE: This is a continuation of the [previous article]({% post_url 2015-05-06-hacking-ubuntu-touch-part-2-devices-and-images %}) in the series. Thanks to Barry Warsaw, Oliver Grawert, Jani Monoses and Ondrej Kubik from Canonical for their assistance.*


We now know what an image is and what it looks like on the inside, but at this point you may wonder: How do all these binary blobs, the ext4 filesystem image and the custom tarball get flashed into device memory? What does ubuntu-device-flash do with them and what happens during the process?

Let's look at what happens after I use `ubuntu-device-flash` to completely reset my bq Aquaris E4.5 Ubuntu Edition to its pristine state using the following command ([here]({% post_url 2015-05-05-hacking-ubuntu-touch-part-1-ubuntu-device-flash %}) I explain the necessity of the `--recovery-image` parameter for this device):


{% highlight text %}
$ ubuntu-device-flash touch --bootstrap --wipe --recovery-image /tmp/recovery.img
{% endhighlight %}

Some of the actions, e.g the upload of the signing keys and tarballs, actually run in parallel, but I list everything in the order in which the processes are spawned, and I assume that the action described in the current step has been successfully completed before the next step begins.


## Preparing the environment


1. `ubuntu-device-flash` uses the `fastboot getvar product` command to detect the attached device. This process blocks until the device is in fastboot mode. Once it shows up, `ubuntu-device-flash` starts downloading the image for the detected device (if necessary).

2. When the device is ready and the image has been successfully downloaded, `fastboot flash recovery /tmp/recovery.img` is spawned to upload the specified temporary recovery image to the internal recovery partition of the phone. This takes about two seconds on my device.

3. `fastboot format cache` is spawned to format the internal cache partition of the device. This takes about two seconds.

4. `fastboot boot /tmp/recovery.img` is spawned to upload the specified temporary recovery image to the device and boot it once.

5. The device boots into the temporarily uploaded recovery image.



## Uploading the image files


The device has restarted, rebooted into Recovery Mode and runs an ADB server.

1. `ubuntu-device-flash` repeatedly calls `adb shell ls` until this is successfull and returns the right directory entries, indicating that the device has booted into recovery.

2. `adb shell rm -rf /cache/recovery/*.xz /cache/recovery/*.xz.asc` is spawned to delete any old image files from the cache partition.

3. `adb push /home/sturmflut/.cache/ubuntuimages/ubuntu-touch/stable/krillin/version-21.tar.xz` is spawned to push the image metadata file to the phone.

4. `adb push /home/sturmflut/.cache/ubuntuimages/gpg/image-signing.tar.xz` is spawned to push the image signing keys to the device. This archive contains the following two keys:

5. `adb push /home/sturmflut/.cache/ubuntuimages/pool/device-0142302186687e3e48e6987283f6caf5d471a4160f98aa6a3cb7658f96471297.tar.xz` is spawned to push the device tarball to the device.

6. `adb push /home/sturmflut/.cache/ubuntuimages/gpg/image-master.tar.xz` is spawned to push the image master keys to the device.

7. `adb push /home/sturmflut/.cache/ubuntuimages/pool/custom-80897f6b2dbcbc12a01735e0fc965a6cc2cdb346f1da05b919406a7790565523.tar.xz` is spawned to push the custom tarball to the device.

8. `adb push /home/sturmflut/.cache/ubuntuimages/pool/ubuntu-d2dfa371c65640e688fd9272b3ede7dbddbfed27f548a0d988c083b1d1c78158.tar.xz` is spawned to push the ubuntu tarball to the device.

9. A temporary commands file is generated and uploaded to the phone as `/cache/recovery/ubuntu_command` via `adb push`.

10. `adb reboot recovery` is spawned to reboot the phone into the temporarily uploaded recovery system.



## The Recovery Mode environment

At this point the phone reboots a second time into Recovery Mode and the recovery init system starts `/sbin/recovery` as always, but this time there's a difference: it detects the `/cache/recovery/ubuntu_command` commands file uploaded earlier and spawns `/sbin/system-image-upgrader` instead of showing the usual user menu. The Ubuntu circle will start spinning on the display and flashing begins.

Maybe we should have a look a the environment before we progress, especially the flash partitions and filesystems information that has been dumped to `/tmp/recovery.log`:

{% highlight text %}
Partition Information:
preloader    0x0000000001400000   0x0000000000000000   2   /dev/misc-sd
mbr          0x0000000000080000   0x0000000000000000   2   /dev/block/mmcblk0
pro_info     0x0000000000300000   0x0000000000100000   2   /dev/block/mmcblk0
nvram        0x0000000000500000   0x0000000000400000   2   /dev/block/mmcblk0
seccfg       0x0000000000020000   0x0000000001d00000   2   /dev/block/mmcblk0
uboot        0x0000000000060000   0x0000000001d20000   2   /dev/block/mmcblk0
bootimg      0x0000000001400000   0x0000000001d80000   2   /dev/block/mmcblk0
recovery     0x0000000001400000   0x0000000003180000   2   /dev/block/mmcblk0
misc         0x0000000000080000   0x0000000004b80000   2   /dev/block/mmcblk0
logo         0x0000000000300000   0x0000000004c00000   2   /dev/block/mmcblk0
expdb        0x0000000000a00000   0x0000000004f00000   2   /dev/block/mmcblk0
bmtpool      0x0000000000000000   0x00000000febf00a8   2   /dev/block/mmcblk0
ebr1         0x0000000000080000   0x0000000000080000   2   /dev/block/mmcblk0p1
protect_f    0x0000000000a00000   0x0000000000900000   2   /dev/block/mmcblk0p2
protect_s    0x0000000000a00000   0x0000000001300000   2   /dev/block/mmcblk0p3
sec_ro       0x0000000000600000   0x0000000004580000   2   /dev/block/mmcblk0p4
cache        0x000000002bc00000   0x0000000005900000   2   /dev/block/mmcblk0p5
android      0x0000000080c00000   0x0000000031500000   2   /dev/block/mmcblk0p6
usrdata      0x000000011e200000   0x00000000b2100000   2   /dev/block/mmcblk0p7
{% endhighlight %}

The first hexadecimal value after the partition name indicates the size of the partition and the second value is the start offset in bytes. The last partition ends at 0x1D02FFFFF, which is 7427 megabytes in, so not all of the 8 gigabytes of flash memory in the device are accessible.

Notice that the first seven partitions point to the same block device, `mmcblk0`. Why? Because they are not part of the partition table, but are special partitions which have a fixed location and a fixed size, depending on how the manufacturer designed the hardware. The kernel creates a character device for each so you don't have to fiddle with offsets:

{% highlight text %}
$ ls -la /dev/

crw-r-----  1 root   system        237,   0 May  6 12:50 preloader
crw-------  1 root   root          237,   1 May  6 12:50 mbr
crw-------  1 root   root          237,   2 May  6 12:50 ebr1
crw-rw----  1 root   system        237,   3 May  6 12:50 pro_info
crw-rw----  1 root   system        237,   4 May  6 12:50 nvram
crw-------  1 root   root          237,   5 May  6 12:50 protect_f
crw-------  1 root   root          237,   6 May  6 12:50 protect_s
crw-rw----  1 root   system        237,   7 May  6 12:50 seccfg
crw-------  1 root   root          237,   8 May  6 12:50 uboot
crw-r-----  1 root   system        237,   9 May  6 12:50 bootimg
crw-r-----  1 root   system        237,  10 May  6 12:50 recovery
crw-r-----  1 root   system        237,  11 May  6 12:50 sec_ro
crw-rw----  1 root   system        237,  12 May  6 12:50 misc
crw-------  1 root   root          237,  13 May  6 12:50 logo
crw-------  1 root   root          237,  14 May  6 12:50 expdb
crw-------  1 root   root          237,  15 May  6 12:50 cache
crw-------  1 root   root          237,  16 May  6 12:50 android
crw-------  1 root   root          237,  17 May  6 12:50 usrdata
crw-------  1 root   root          237,  18 May  6 12:50 bmtpool
{% endhighlight %}


Of all these partitions, only three (the first entry is a ramdisk and the "emmc" ones are fake devices) are used as actual filesystems, namely `mmcblk0p6` and `mmcblk0p7`:


{% highlight text %}
recovery filesystem table
=========================
  0 /tmp ramdisk (null) (null) 0
  1 /uboot emmc /dev/uboot (null) 0
  2 /boot emmc /dev/bootimg (null) 0
  3 /cache ext4 /dev/block/mmcblk0p5 (null) 0
  4 /data ext4 /dev/block/mmcblk0p7 (null) 0
  5 /misc emmc misc (null) 0
  6 /recovery emmc /dev/recovery (null) 0
  7 /system ext4 /dev/block/mmcblk0p6 (null) 0
{% endhighlight %}




## The "system-image-upgrader does the work" part


`system-image-upgrader` interprets the commands from the `/cache/recovery/ubuntu_command` commands file. I've written about this file a couple of times already and now is the moment to look at it.

{% highlight text %}
~ # cat /cache/recovery/ubuntu_command.applying

format data
format system
load_keyring image-master.tar.xz image-master.tar.xz.asc
load_keyring image-signing.tar.xz image-signing.tar.xz.asc
mount system
update ubuntu-d2dfa371c65640e688fd9272b3ede7dbddbfed27f548a0d988c083b1d1c78158.tar.xz ubuntu-d2dfa371c65640e688fd9272b3ede7dbddbfed27f548a0d988c083b1d1c78158.tar.xz.asc
update device-0142302186687e3e48e6987283f6caf5d471a4160f98aa6a3cb7658f96471297.tar.xz device-0142302186687e3e48e6987283f6caf5d471a4160f98aa6a3cb7658f96471297.tar.xz.asc
update custom-80897f6b2dbcbc12a01735e0fc965a6cc2cdb346f1da05b919406a7790565523.tar.xz custom-80897f6b2dbcbc12a01735e0fc965a6cc2cdb346f1da05b919406a7790565523.tar.xz.asc
update version-21.tar.xz version-21.tar.xz.asc
unmount system
{% endhighlight %}


This is pretty much what one would expect, but I'll explain the steps:

1. The `data` and `system` partitions are formated as ext4 filesystems.

2. The GPG keyrings are loaded and validated against their signature files, which are signed by GPG key IDs 3F272F5B and D265EACD (the Ubuntu master signing keys).

3. the `system` partition is mounted.

4. The `ubuntu` archive is unpacked to `/` and validated against its signature. If you've wondered why all the contents of the archive is in a `system/` sub-directory, this is why: It has to end up in `/system` inside the Recovery  Environment.

5. The `device` archive is unpacked to `/` and validated against its signature. We've looked at this archive in the previous article, and we know that it contains binary blob images (`uboot.img`, `recovery.img` and `boot.img`) besides an Android bit container. It will probably be surprising for many that those binary blobs are written to their respective partitions in this step as well! So the `unpack` command also includes flashing!

6. The `custom` archive is unpacked to `/` and validated against its signature.

7. The `version` archive is unpacked to `/` and validated against its signature. I am not sure why this is necessary, as we've seen in the previous article that it only contains two small metadata files.

8. The `system` partition is unmounted.

At this time the phone is fully flashed and the system reboots into normal mode.

If you know better and/or something has changed, please find me on Launchpad.net and do get in contact!