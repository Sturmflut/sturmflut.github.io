---
layout: post
title:  "Installing Ubuntu on BayTrail tablets (version 2)"
date:   2015-02-04 18:00:00
categories: Linux Ubuntu
---

This article is based on my [previous version]({% post_url 2015-01-21-installing-ubuntu-15.04-on-baytrail-tablets %}). I no longer use Ubuntu 15.04, but 14.10, because 15.04 switched to the 3.18 kernel which is incompatible with the eMMC flash in my tablet.


## Requirements


- A BayTrail tablet, probably running Windows 8.1. I got a Lenovo ThinkPad Tablet 8 for the special price of 180 € from a local retailer.
- An USB-On-the-Go-Adapter. USB 2.0 will work, but you really want USB 3.0.
- A powered (!) USB hub. USB 2.0 will work, but you really want USB 3.0. If the hub is not powered, be prepared for USB resets or even a complete disconnect every time you connect a new device, because the tablet may not be able to supply enough power.
- An USB storage medium to boot from.
- An USB keyboard and USB Mouse.
- An USB network adapter, either Ethernet or WiFi, supported by Ubuntu 14.10, if the built-in WiFi of your device doesn't work.
- Optional: A storage medium (USB, SD-Card etc.) big enough to back up the Windows installation. I used a 64 GB SD-Card.


## Current status on the Thinkpad Tablet 8

What works:

- CPU initialisation and SMP (all four cores are working)
- GPU initialisation, accelerated 2D, 3D and video
- Touchscreen
- Backlight
- USB 3.0 On-the-go

What doesn't work:

- ACPI battery detection: The battery is not detected.
- WiFi: The brcmfmac driver needs you to create a configuration file named `/lib/firmware/brcm/brcmfmac43241b4-sdio.txt`. The content of this file can be extracted from the UEFI vars, at least on my tablet the variable is called `/sys/firmware/efi/efivars/nvram-74b00bd9-805a-4d61-b51f-43268123d113`. Everything is present, the brcmfmac driver is loaded and doesn't complain, but at the end the kernel does not report any wireless devices.
- Bluetooth: Is bundled with the WiFi.
- Sound: Was not working and I didn't try to fix it.
- The two cameras: Connected via some obscure bus (MIPI-CSI) which isn't supported.


## A word about kernels

I couldn't get any of the stable kernels from kernel.org working because they have problems with the internal eMMC flash in my ThinkPad Tablet 8 (see [this emailing list discussion][gmane-linux-mmc-thinkpad-tablet-10]). After at most 20 minutes the kernel freezes and the last message is `"mmc0: Timeout waiting for hardware interrupt"`.

Ubuntu 14.04 and 14.10 kernels work, obviously because they have been patched. At the moment I don't know exactly what has to be patched. The Ubuntu 15.04 kernel does not work.


## Installation


### Disabling Secure Boot using Windows

1. Boot into Windows.
2. Go into "PC settings".
3. Select "General" (Windows 8) or "Update and recovery" (Windows 8.1).
4. Under "Advanced startup", click on "Restart now".
5. At this point I'm not completely sure how the story went, and I can no loner check, since the Windows installation on my tablet is already gone. You are looking for some way to get into the UEFI firmware settings, probably hidden behind Items like "Troubleshoot", "Advanced options" etc.
6. Once you are in the UEFI menu, disable Secure Boot.
7. Set the Boot Order so that external USB drives are at the top and listed before the internal storage. Different manufacturers seem to use different UEFI firmwares with different menus, so you have to find the options yourself.
8. Power off.

The tablet is now prepared.


### Disabling Secure Boot by directly booting into the UEFI menu

At least the ThinkPad Tablet 8 can be directly booted into UEFI by powering off the device and then pressing the Volume-Up- and Power-button until the display turns on. If you press the buttons for too long, the tablet will turn off again!

Once you are in the UEFI menu, disable Secure Boot. Set the Boot Order so that external USB drives are at the top and listed before the internal storage. Different manufacturers seem to use different UEFI firmwares with different menus, so you have to find the options yourself.

The tablet is now prepared.


## Preparing the Ubuntu 14.10 installation medium

*** NOTE: In my earlier version I suggested to download the latest version of 

1. Download the most recent 64-bit [Ubuntu 14.10 (Utopic Unicorn) Desktop ISO][utopic-desktop-iso].
2. Create a bootable USB storage medium as usual, e.g. using usb-creator-gtk.


## Modifying the installation medium to boot on 32-bit UEFI

Yes, some genius came up with the idea to put a 32-bit UEFI on a 64-bit device. The Linux kernel shipped with Ubuntu 14.10 supports mixed-mode, so kernel-wise it is no longer a problem. But the Ubuntu 14.10 (Vivid Vervid) ISO does not contain a 32-bit bootloader. Luckily this can be easily fixed.

1. Get a `bootia32.efi` file, I used [this one][bootia32-efi] from John Wells.
2. Copy the file into the `EFI/BOOT/` folder on the boot medium.


## Booting from the modified boot medium

1. Power off the tablet, if necessary.
2. Attach the USB-On-the-Go-Adapter to the tablet.
3. Connect the USB hub to the adapter.
4. Connect the boot medium to the USB hub.
5. Connect the keyboard and mouse to the hub.
6. Start the tablet.
7. The device should boot from the boot medium.

If you can't get the tablet to boot from the USB storage medium, try booting into the UEFI menu again and check if it is possible to enable a Boot Device selection screen. The Thinkpad Tablet 8 has an option to boot into such a screen if the F12 key is pressed after device power-on.


## Backing up the Windows installation

I just started a terminal in the Live Session and used `dd` to backup all internal flash partitions to individual files on an SD-Card.


## Installing Ubuntu 14.10 on the device

1. Start a terminal session and switch do root (using `sudo -i` or something similar)
2. Run `parted`
3. Preserve the first two partitions (EFI and Windows Recovery) and delete all other partitions
4. Create a large ext4 partition in the remaining space
5. Create a small swap partition in the remaining space (e.g. 4 GB)
6. Write to disk and exit

I don't trust the flash storage, so I created the filesystem manually and told mkfs.ext4 to check for bad blocks:

`mkfs.ext4 -v -c /dev/mmcblk0p3`

After this I just used the Ubuntu Installer, chose "Partition manually" and configured as follows:

1. The EFI partition (should be the first one) shall be used as EFI partition
2. The just-created ext4 partition (should be the third one) shall be used as the root filesystem. Deactivate the "Format" checkbox if it is set.
3. The swap partition shall be used as a swap partition.



## Installing 32-bit GRUB

Ubuntu 15.04 comes with a 32-Bit GRUB package, but 14.10 does not, so we have to do it the hard way.

The following steps are common to both methods:

1. Reboot the device, with the USB storage medium plugged in.
2. Wait until the device has booted into GRUB from the USB storage medium.
3. Hit "c" on the keyboard to drop to the GRUB shell.
4. Input the following two lines, adapting all device and partition identifiers and kernel versions to your setup (GRUB has autocompletion!):

{% highlight bash %}
linux (hd0,gpt5)/boot/vmlinuz-3.16.0-23-generic root=/dev/mmcblk0p3
initrd (hd0,gpt5)/boot/initrd-3.16.0-23-generic
{% endhighlight %}

5. Boot with the command "boot".
6. If the system doesn't boot, go back to step 1.
7. Log into the Ubuntu installation.
8. Connect to the internet using the internal WiFi or your USB network adapter.

### Doing it the hard way

Reboot and manually boot into your new installation:

1. Open a terminal and install the dependencies:

{% highlight bash %}
$ apt-get install autoconf bison flex
{% endhighlight %}


2. Open a terminal and check out the GRUB git repository:

{% highlight bash %}
$ git clone git://git.savannah.gnu.org/grub.git
{% endhighlight %}

3. Build a 32-bit GRUB:

{% highlight bash %}
$ cd grub
$ ./autogen.sh
$ ./configure --with-platform=efi --target=i386 --program-prefix=""
$ make -j6
{% endhighlight %}

4. Install 32-bit grub:

{% highlight bash %}
$ cd grub-core
$ sudo ../grub-install -d . --efi-directory /boot/efi/ --target=i386
{% endhighlight %}

5. Replace the Ubuntu bootloader with our GRUB one, making sure that there are only the listed files in the directory:

{% highlight bash %}
$ cd /boot/efi/EFI/
$ sudo cp grub/grubia32.efi ubuntu/grubx64.efi

$ ls ubuntu
grub.cfg  grub.efi  grubx64.efi  MokManager.efi  shimx64.efi
{% endhighlight %}

6. Update the grub config:

{% highlight bash %}
$ sudo update-grub
{% endhighlight %}

You can set the UEFI boot order using the `efibootmgr` command, but I am not completely sure as to what the correct settings are. The following settings were generated by `update-grub` and seem to works on my tablet:

{% highlight bash %}
$ efibootmgr 
BootCurrent: 0001
Timeout: 1 seconds
BootOrder: 0000,0001,0014,0015,0016,0017,0013,0018
Boot0000* ubuntu
Boot0001* grub
Boot0010  Setup
Boot0011  Boot Menu
Boot0012  System Recovery
Boot0013* Internal Storage:
Boot0014* USB HDD:
Boot0015* USB CD/DVD:
Boot0016* USB FDD:
Boot0017* CD-ROM:
Boot0018* Network Adapter:
{% endhighlight %}



### The probably much better way (only available on Ubuntu 15.04)

1. Open a terminal and install the `grub-efi-ia32-bin` package:

{% highlight bash %}
$ sudo apt-get install grub-efi-ia32-bin
{% endhighlight %}

2. Update the grub config:

{% highlight bash %}
$ sudo update-grub
{% endhighlight %}


### The end

Now your device should boot into the Ubuntu installation on every boot. If it doesn't, try to fiddle with the boot order list. You can always boot into your installation by booting from the external USB storage medium and going through the GRUB console, as shown before.


As always you can find me on the FreeNode IRC.


[gmane-linux-mmc-thinkpad-tablet-10]: http://www.spinics.net/lists/linux-mmc/msg30631.html

[utopic-desktop-iso]: http://releases.ubuntu.com/14.10/

[linux-mmc-sebastien]: http://article.gmane.org/gmane.linux.kernel.mmc/31030

[jwells-ubuntu-t100]: http://www.jfwhome.com/2014/03/07/perfect-ubuntu-or-other-linux-on-the-asus-transformer-book-t100/

[bootia32-efi]: https://github.com/jfwells/linux-asus-t100ta/blob/master/boot/bootia32.efi

[softpedia]: http://news.softpedia.com/news/Ubuntu-Touch-Spotted-Running-on-Former-Windows-8-1-Tablet-Lenovo-ThinkPad-8-469594.shtml
