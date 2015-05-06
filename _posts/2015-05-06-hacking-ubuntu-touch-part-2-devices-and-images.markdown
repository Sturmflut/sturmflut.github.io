---
layout: post
title:  "Hacking Ubuntu Touch, Part 2: Devices and Images"
date:   2015-05-06 20:00:00
categories: Ubuntu Touch
---

*NOTE: This is a continuation of the [previous article]({% post_url 2015-05-05-hacking-ubuntu-touch-part-1-ubuntu-device-flash %}) in the series.*

We now know how to work with `ubuntu-device-flash` to find, download and flash images. But what does an image actually contain? Let's look at the information for the most recent image for the bq Aquaris E4.5 Ubuntu Edition as obtained in the previous article:


{% highlight text %}
$ ubuntu-device-flash query --device=krillin --channel=ubuntu-touch/rc/bq-aquaris.en --show-image
Device: krillin

Description: ubuntu=20150410.1,device=20150408-4f14058,custom=20150409-665-29-206,version=22
Version: 22

Channel: ubuntu-touch/rc/bq-aquaris.en
Files:
 0 https://system-image.ubuntu.com/pool/ubuntu-d2dfa371c65640e688fd9272b3ede7dbddbfed27f548a0d988c083b1d1c78158.tar.xz 300609184 e7165fc250eb960bc519ff3e564701a6abd49c32aaa3144b32411881c2bf1eec                                                                       
 1 https://system-image.ubuntu.com/pool/device-0142302186687e3e48e6987283f6caf5d471a4160f98aa6a3cb7658f96471297.tar.xz 70683616 03603360f1259d57c4cd7fdab3624ea57350b01647062de7d229aa2d0df4e17b
 2 https://system-image.ubuntu.com/pool/custom-80897f6b2dbcbc12a01735e0fc965a6cc2cdb346f1da05b919406a7790565523.tar.xz 63334536 aee001c435d14c7a52c1cf63ff37771a84ecca9ecb85a2ac6f9d7aaaf403cc3c
 3 https://system-image.ubuntu.com/ubuntu-touch/rc/bq-aquaris.en/krillin/version-22.tar.xz 376 424fc2eb690ad52229ffc66120bf3d46d5398909d0ce53cbfc1b4513193a993e
{% endhighlight %}


The image consists of four actual files: an `ubuntu` tarball, a `device` tarball and a `custom` tarball. The `version` archive just contains a bit of metdata:


{% highlight text %}
$ tar -xvf version-22.tar.xz

system/etc/ubuntu-build
system/etc/system-image/channel.ini

$ cat system/etc/ubuntu-build
22

$ cat system/etc/system-image/channel.ini
[service]

base: system-image.ubuntu.com
http_port: 80
https_port: 443
channel: ubuntu-touch/rc/bq-aquaris.en
device: krillin
build_number: 22
version_detail: ubuntu=20150410.1,device=20150408-4f14058,custom=20150409-665-29-206,version=22
{% endhighlight %}


So an Ubuntu Touch archive consists of three archives which go together somehow, and this is the layering, showing the different parts used to build images for different devices:


{% highlight text %}
----------------      -------------     ----------------
| Nexus custom |      | bq custom |     | Meizu custom |
----------------      -------------     ----------------

--------------------------------------------------------
|                   Ubuntu 15.04 base                  |
--------------------------------------------------------

------------------    ---------------     --------------
| Nexus 4 device |    | E4.5 device |     | MX4 device |
------------------    ---------------     --------------
{% endhighlight %}


Let's look at what the individual tarballs contain.



## Device tarball

Phones and tablets are different from personal computers and servers because they are embedded platforms, which are usually short on memory and processing power. Also the hardware setup is pretty much the same all the time and doesn't change, with the exception of USB, but the number of supported USB devices is also limited. So it doesn't make sense to enable every possible kernel feature and ship every available driver like the normal Ubuntu Desktop and Server flavors do.

On non-x86 platforms (especially ARM) a second problem arises: There is no standardised way to boot a system and to find out what the hardware looks like. So you can't build a generic kernel that would run on more than one ARM board at the same time because every board is different from the others and the kernel has no way to find out what to do - there's no BIOS, UEFI, DMI, ACPI etc. You have to manually craft a matching kernel for the exact board you're working with, and in many cases you also have to craft a matching bootloader (usually U-Boot) for the exact board. There are some solutions for this, namely DeviceTree, and future ARM64 systems will support UEFI and ACPI. But Android has set a quasi-standard for these environments and most Ubuntu Touch devices will just be repurposed Android models, so it will probably be a long time before our phones look like our personal computers and we can just install a generic, non-modified Ubuntu on any device without having to think about the hardware.

Now what does the device tarball contain?


{% highlight text %}
$ tar -tvf device-0142302186687e3e48e6987283f6caf5d471a4160f98aa6a3cb7658f96471297.tar.xz 

-rw-rw-r-- root/root    247968 2015-04-08 17:31 partitions/uboot.img
-rw-r--r-- root/root   8605696 2015-04-08 17:31 partitions/recovery.img
-rw-r--r-- root/root   9523200 2015-04-08 17:31 partitions/boot.img
-rw-rw-r-- root/root        17 2015-04-08 17:31 system/etc/device-build
-rw-rw-r-- root/root 151080960 2015-04-08 17:31 system/var/lib/lxc/android/system.img
{% endhighlight %}


There are four binary files:

* `uboot.img` is the U-Boot bootloader for the device. This is a binary blob.

* `recovery.img` is the recovery system image that gets booted when you enter [Recovery Mode]({% post_url 2015-05-04-hacking-the-bq-part-1-bootloader-fastboot-recovery %}). This is a blob in the "Android bootimg" format which will be discussed in a later article.

* `boot.img` contains the kernel and initrd that get booted when your device boots normally or when you enter [Factory Mode]({% post_url 2015-05-04-hacking-the-bq-part-2-factory-mode %}). This also is a blob in the "Android bootimg" format.

* `system.img` contains the Android bits for the device. This is necessary because many manufacturers only provice Android drivers for their hardware and Ubuntu Touch needs them to work, interfacing via [libhybris][libhybris]. This is an ext4 filesystem image and its content will later be launched in an LXC container.


You can mount `system.img` on a host computer and look at its content:


{% highlight text %}
$ mkdir /tmp/android-system

$ mount -o loop,ro system.img /tmp/android-system

$ find /tmp/android-system
{% endhighlight %}

It is pretty large, currently about 120 megabytes, and contains a lot of files.



## Ubuntu tarball


This tarball contains the actual Ubuntu Touch base system with all its default libraries, frameworks, apps, services and system binaries. This looks very much like a normal Ubuntu system, and in fact it is. AppArmor, Upstart, udev, NetworkManager, Qt5, bash? There. Unity? There. There are also many additional, phone-specific bits like ofono and system-image.

There are currently about 44.000 files in there, so if you're interested just use an archiver and have a look.



## Custom tarball


Canonical promised their partners that an Ubuntu Touch device can be customised to set each device apart from the others. This enables a manufacturer to ship additional files, apps and scopes with the default installation of a device, for example the bq Aquaris E4.5 Ubuntu Edition came with the Nokia HERE location provider (codenamed "espoo"), the Aquaris ringtone, a "NearBy" scope and a free copy of the "Cut the Rope" game, all goodies currently not available on other phones.

In the end it's just a set of files which are extracted to the internal device memory after all other steps have been finished, extending the default installation.

Did you know that you can just bundle your own `custom` tarball, and that `ubuntu-device-flash touch` has a `--custom-tarball=` parameter? This is where your modding skills come in, kids! I wonder if someone could make a "super" custom tarball, containing all the goodies from all device manufacturers at the same time...





[libhybris]: https://en.wikipedia.org/wiki/Hybris_%28software%29