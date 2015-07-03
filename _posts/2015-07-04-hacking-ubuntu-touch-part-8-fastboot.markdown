---
layout: post
title:  "Hacking Ubuntu Touch, Part 8: Fastboot"
date:   2015-07-04 04:00:00
categories: Ubuntu Touch
---

*NOTE: This is a continuation of the [series]({% post_url 2015-05-07-hacking-ubuntu-touch-index %}).*

*NOTE: The information in this article was obtained by analzying the reference implementation client and disecting the protocol traffic with Wireshark. The `signature` feature of the reference client seems to be undocumented, most likely for a good reason. Don't take this as a reference!*


Sometimes you need direct access to the lower levels of your device, and we have already looked at how [ADB]({% post_url 2015-05-08-hacking-ubuntu-touch-part-4-developer-mode-and-ADB %}) enables developers and power users to do many useful things. But what do you do when you can't rely on the operating system, which runs the ADB daemon? Maybe you want to push and boot a modified kernel just once? Or your OS is broken and no longer exposes an ADB interface, so you have to re-flash everything?

This is where Fastboot comes in.



## Fastboot basics

Fastboot is a simple interface to the Android bootloader. It is much more generic than vendor-specific protocols (look at all the crazy solutions in the [MediaTek]({% post_url 2015-07-02-mediatek-details-soc-startup %}) [articles]({% post_url 2015-07-04-mediatek-details-partitions-and-preloader %})), but allows vendor-specific extensions. Most Android device bootloaders can be switched to Fastboot mode, but not every device might interpret every possible command. A "locked" bootloader e.g. will typically refuse to flash partitions or only let signed images through. Sometimes it may be possible to unlock the bootloader.

Unlike ADB, which also runs over TCP/IP, Fastboot only supports USB. The design is much simpler, there are only the bootloader and a client on the USB Host. The reference client implementation is part of the Android SDK.




## The Fastboot wire protocol

The protocol is very simple: It consists of ASCII text commands up to 64 bytes in size, in the form `<command>:<parameter>`. All data is transported in USB Bulk mode frames. The bootloader will answer every command with an ASCII text response of up to 64 bytes in length. The following responses are legal:

* `INFO`, potentially followed by an informational message that the reference client prints to `stderr`.

* `OKAY`, potentially followed by a response that may or may not be checked.

* `FAIL`, potentially followed by a failure reason.

* `DATA`, followed by the amount of data the bootloader is willing to accept. This is always compared against the amount of data the client announced as a parameter with the command, and the command is aborted if it wants to send more data than the bootloader can take.

If the command has a payload, it is sent in a second transfer after a positive response.


The following commands are used in the reference implementation:

`getvar:<variable>` queries a bootloader variable. Every bootloader may expose different variables, the following ones are actively used by the reference client and exposed by the MediaTek Little Kernel:

* `partition-type:<name>` returns the type of the partition with the given name, e.g. "raw data" or "ext4".

* `partition-size:<name>` returns the size of the partition with the given name in hexadecimal.

* `max-download-size` returns the maximum amount of data that can be written to the device using the `download` command, if there is a limit.

* `product` returns the product name, e.g. "krillin" on the bq Aquaris E4.5 Ubuntu Edition.

* `kernel` returns "lk" on the bq Aquaris E4.5 Ubuntu Edition.

* `version` returns "0.5" on the bq Aquaris E4.5 Ubuntu Edition.

`erase:<name>` erases the partition with the given name.

`download:<size>` sends `size` (in hexadecimal, max. 32 bits) bytes of data to the bootloader memory for later processing.

`flash:<name>` writes the data the bootloader has previously received using the `download` command to the partition with the given name. Note that if multiple pairs of the `download` and `flash` commands are issued to the same partition, every `flash` command will append its data at the end of the previous writes. This means that large partitions have to be written in blocks of `max-download-size` bytes.

`signature` activates a 256 bytes long signature that was previously sent to the device using the `download` command. I am not sure what the bootloader does with this signature, probably it is uses for a cryptographic verification process.

`continue` aborts Fastboot mode and the bootloader continues to boot normally.

`boot` boots into an Android boot image that was previously sent to the device using the `download` command.

`reboot-bootloader` reboots into the bootloader.




## OEM commands

The bootloader may implement additional, manufacturer-specific commands. These do not follow the usual protocol command format, and they cannot have a payload. Some manufacturers implement a *lot* of OEM commands, as can be seen [here][xda-fastboot-oem]. MediaTek just implements two: `fastboot oem p2u on` and `fastboot oem p2u off`, these enable or disable the redirection of Linux kernel messages to the UART.

Note that the most well-known OEM command, `fastboot oem unlock`, is *not* available on the most devices. This command was invented by Google for their Nexus devices.



## Reference client commands

The reference fastboot client supports the following commands:

* `devices`: Detects and lists attached devices.

* `getvar <variable>`: Issues the `getvar` wire command with the given variable.

* `format <partition>`: Formats the given partition. The client will first request the `partition-type` of the partition and compare it against a list of known "generators" (mkfs tools), of which there currently is just one: ext4. If there is a generator for the returned partition type, the `partition-size` is requested and an empty filesystem image generated on `/tmp` on the host. This image is then downloaded to device memory and flashed to the given partition.

* `signature <file>`: The given signature file is downloaded to the device and activated using the `signature` command.

* `reboot`: Issues the `reboot` wire command.

* `reboot-bootloader`: Issues the `reboot-bootloader` wire command.

* `continue`: issues the `continue` wire command.

* `boot <kernel> [<ramdisk>]`: Loads the given kernel image file (and optionally the ramdisk), builds an Android boot image on the fly, downloads the generated image to device memory and issues the `boot` wire command.

* `flash <partition> [<filename>]`: Checks the `max-download-size` the device supports, then downloads the given image file to the partition with the given name in chunks ("sparse mode", if necessary). If no filename is given, the client tries to guess the filename from a list of default filenames (`boot.img`, `recovery.img` etc.) and the product name.

* `flash:raw <partition> <kernel> [<ramdisk>]`: Loads the given kernel image file (and optionally the ramdisk), builds an Android boot image on the fly, downloads the generated image to device memory and flashes it to the given partition.

* `flashall`: This is probably intended for automated flashing on production lines. The client reads key-value pairs from a configuration file called `android-info.txt` and requests every key using the `getvar` command. The responses are compared to the values from the configuration file, making sure that the device is in the expected state. Then `boot.img`, `recovery.img`, `system.img` and their associated signature files are downloaded, activated and written to the corresponding partitions.

* `update <filename>`: Applies a device update from a ZIP archive. This is exactly the same step as `flashall`, except that the image and signature files are extracted from the archive.

* `oem <arguments..>`: Sends an OEM command.

The reference client supports the concatenation of commands, e.g. the following works:


{% highlight text %}
$ fastboot getvar kernel getvar version
kernel: lk
version: 0.5
{% endhighlight %}

The following command line options are supported:

* `-w`: Erase and format the `userdata` and `cache` partitions before executing any other commands.

* `-u`: Erase partitions before flashing them.

* `-b <addr>`: Set an alternative base address for the `boot` and `flash:raw` commands.

* `-n <size>`: Set an alternative page size for the `boot` and `flash:raw` commands (default: 2048).

* `-s <serial>`: Set a target device serial number in case multiple devices are connected. Can also be set through the `ANDROID_SERIAL` environment variable.

* `-S <limit>`: Overrides the the `max-download-size` limit reported by the device.

* `-l`: List device paths when executing the `devices` command.

* `-p <product>`: Set the product name, can also be set through the `ANDROID_PRODUCT_OUT` environment variable.

* `-c <cmdline>`: override the kernel command line used when generating Android boot images on the fly.

* `-i <id>`: Specify a vendor ID in case the client does not detect the USB device.


That's it for today. If you know better and/or something has changed, please do find me on the Freenode IRC or on Launchpad.net and get in contact!



[xda-fastboot-oem]: http://forum.xda-developers.com/showthread.php?t=2300654