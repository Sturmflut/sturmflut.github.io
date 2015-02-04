---
layout: post
title:  "Installing Ubuntu 15.04 on BayTrail tablets"
date:   2015-01-21 18:00:00
categories: Linux Ubuntu
---

***NOTE: This version has been obsoleted by [version 2]({% post_url 2015-02-04-installing-ubuntu-on-baytrail-tablets-version-2 %}).***

I was asked to do a little writeup about how I [installed Ubuntu 15.04 on my Intel BayTrail tablet][softpedia]. I am short on time, so I'll just write down the most basic steps and do a full How-To later.

The following instructions are based on [this article by John Wells][jwells-ubuntu-t100], but many things have changed since march 2014, so not every step listed in the original article is still necessary.


## Requirements


- A BayTrail tablet, probably running Windows 8.1. I got a Lenovo ThinkPad Tablet 8 for the special price of 180 € from a local retailer.
- An USB-On-the-Go-Adapter. USB 2.0 will work, but you really want USB 3.0.
- A powered (!) USB hub. USB 2.0 will work, but you really want USB 3.0. If the hub is not powered, be prepared for USB resets or even a complete disconnect when you connect a new device, because the tablet may not be able to supply enough power by itself.
- An USB storage medium to boot from.
- An USB keyboard and USB Mouse.
- An USB network adapter, either Ethernet or WiFi, supported by Ubuntu 15.04, if the built-in WiFi of your device doesn't work.
- Optional: A storage medium (USB, SD-Card etc.) big enough to back up the Windows installation. I used a 64 GB SD-Card.


## Disabling Secure Boot using Windows

1. Boot into Windows.
2. Go into "PC settings"
3. Select "General" (Windows 8) or "Update and recovery" (Windows 8.1)
4. Under "Advanced startup", click on "Restart now"
5. At this point I'm not completely sure how the story went, and I can no loner check, since the Windows installation on my tablet is already gone. You are looking for some way to get into the UEFI firmware settings, probably hidden behind Items like "Troubleshoot", "Advanced options" etc.
6. Once in the UEFI menu, disable Secure Boot and set the boot order so that external USB drives are at the top and listed before the internal storage. Different manufacturers seem to use different UEFI firmwares with different menues, so you have to find the options yourself.
7. Power off.
8. The tablet is now prepared.


## Disabling Secure Boot by directly booting into the UEFI menu

At least the ThinkPad Tablet 8 can be directly booted into UEFI by powering off the device and then pressing the Volume-Up- and Power-button until the display turns on. If you press the buttons for too long, the tablet will turn off again!


## Preparing the Ubuntu 15.04 installation medium

1. Download the most recent 64-bit Ubuntu 15.04 (Vivid Vervid) ISO.
2. Create a bootable USB storage medium as usual.


## Modify the installation medium to boot on 32-bit UEFI

Yes, some genius came up with the idea to put a 32-bit UEFI on a 64-bit device. The Linux kernel shipped with Ubuntu 15.04 supports mixed-mode, so kernel-wise it is no longer a problem. But the Ubuntu 15.04 (Vivid Vervid) ISO does not contain a 32-bit bootloader. Luckily this can be easily fixed.

1. Get a `bootia32.efi` file, I used [this one][bootia32-efi] from John Wells.
2. Copy the file into the `EFI/BOOT/` folder on the boot medium.


## Booting from the modified boot medium

1. Power off the tablet, if necessary.
2. Attach the USB-On-the-Go-Adapter to the tablet
3. Connect the USB hub to the adapter
4. Connect the boot medium to the USB hub.
3. Connect the keyboard and mouse to the hub.
3. Start the tablet.
4. The device should boot from the boot medium. Boot into the Live Session.


## Backing up the Windows installation

I just started a terminal in the Live Session and used `dd` to backup all internal flash partitions to individual files on an SD-Card.


## Installing Ubuntu 15.04 on the device

I just installed Ubuntu 15.04 using the installer. Manually partition the disk, but preserve the UEFI partition!


## Installing 32-bit GRUB

At this point I am not sure what is currently the right method. I did it the hard way, but have since found out that there should be a better way.

The following steps are common to both methods:

1. Reboot the device, with the USB storage medium plugged in.
2. Wait until the device has booted into GRUB from the USB storage medium.
3. Hit "c" on the keyboard to drop to the GRUB shell.
4. Input the following two lines, adapting all device and partition identifiers and kernel versions to your setup (GRUB has autocompletion!):

{% highlight bash %}
linux (hd0,gpt5)/boot/vmlinuz-3.18.0-9-generic root=/dev/mmcblk0p5
initrd (hd0,gpt5)/boot/initrd-3.18.0-9-generic
{% endhighlight %}

5. Boot with the command "boot".
6. If the system doesn't boot, go back to step 1.
7. Log into the Ubuntu installation.
8. Connect to the internet using the internal WiFi or your USB network adapter.

### Doing it the hard way

Reboot and manually boot into your new installation:

1. Open a terminal and check out the GRUB git repository:

{% highlight bash %}
$ git clone git://git.savannah.gnu.org/grub.git
{% endhighlight %}

2. Build a 32-bit GRUB:

{% highlight bash %}
$ cd grub
$ ./autogen.sh
$ ./configure --with-platform=efi --target=i386 --program-prefix=""
$ make -j4
{% endhighlight %}

3. Install 32-bit grub:

{% highlight bash %}
$ cd grub-core
$ sudo ../grub-install -d . --efi-directory /boot/efi/ --target=i386
{% endhighlight %}

4. Update the grub config:

{% highlight bash %}
$ sudo update-grub
{% endhighlight %}


### The probably much better way

1. Open a terminal and install the `grub-efi-ia32-bin` package:

{% highlight bash %}
$ sudo apt-get install grub-efi-ia32-bin
{% endhighlight %}

2. Update the grub config:

{% highlight bash %}
$ sudo update-grub
{% endhighlight %}


### The end

Now your device should boot into the Ubuntu installation on every boot.


### Issues regarding the Lenovo ThinkPad Tablet 8

Last time I worked on it, the following things did not work properly:

- ACPI battery detection: The battery is not detected.
- WiFi: The brcmfmac driver needs you to create a configuration file in `/lib/firmware/brcm/brcmfmac43241b4-sdio.txt`. I think I've got the right configuration, the firmware file `/lib/firmware/brcm/brcmfmac43241b4-sdio.bin` is present, the brcmfmac driver is loaded and doesn't complain, but at the end the kernel does not report any wireless devices.
- Bluetooth: Is not detected and I have no idea where to look at.
- Sound: Was not working and I didn't try to fix it.

These issues were not fixed in the 3.19-rc3 kernel.


[jwells-ubuntu-t100]: http://www.jfwhome.com/2014/03/07/perfect-ubuntu-or-other-linux-on-the-asus-transformer-book-t100/

[bootia32-efi]: https://github.com/jfwells/linux-asus-t100ta/blob/master/boot/bootia32.efi

[softpedia]: http://news.softpedia.com/news/Ubuntu-Touch-Spotted-Running-on-Former-Windows-8-1-Tablet-Lenovo-ThinkPad-8-469594.shtml
