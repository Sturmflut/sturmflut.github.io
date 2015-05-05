---
layout: post
title:  "Hacking Ubuntu Touch, Part 1: ubuntu-device-flash"
date:   2015-05-05 22:00:00
categories: Ubuntu Touch
---


The [Ubuntu Touch Install][ubuntu-touch-install] documentation shows how to flash Ubuntu Touch to a supported device using the `ubuntu-device-flash` command. Let's have a look at what else this command can do.


## Query the image server

List available channels for the bq Aquaris E4.5 Ubuntu Edition (krillin):

{% highlight text %}
$ ubuntu-device-flash query --device=krillin --list-channels

ubuntu-touch/devel-proposed (alias to ubuntu-touch/vivid-proposed)
ubuntu-touch/rc/bq-aquaris.en
ubuntu-core/15.04/edge
ubuntu-touch/devel (alias to ubuntu-touch/vivid)
ubuntu-touch/devel-proposed-customized-here (alias to ubuntu-touch/vivid-proposed-customized-here)
ubuntu-touch/devel-proposed/krillin.en-proposed
ubuntu-touch/ubuntu-rtm/devel-customized (alias to ubuntu-touch/ubuntu-rtm/14.09-customized)
ubuntu-touch/vivid
ubuntu-core/15.04/stable
ubuntu-core/devel-proposed
ubuntu-core/rolling/alpha
ubuntu-core/rolling/edge
ubuntu-touch/rc/ubuntu-developer
ubuntu-touch/stable/ubuntu-developer
ubuntu-touch/ubuntu-rtm/14.09
ubuntu-touch/devel-proposed/krillin.en
ubuntu-touch/stable/bq-aquaris.en (alias to ubuntu-touch/ubuntu-rtm/14.09)
ubuntu-touch/stable/meizu.en
ubuntu-touch/ubuntu-rtm/14.09-customized
ubuntu-touch/ubuntu-rtm/14.09-factory-proposed
ubuntu-touch/vivid-proposed-customized-here
ubuntu-core/devel
ubuntu-touch/devel-proposed/ubuntu-developer
ubuntu-touch/stable (alias to ubuntu-touch/ubuntu-rtm/14.09)
{% endhighlight %}



List available images for the ubuntu-touch/rc/bq-aquaris.en channel:



{% highlight text %}
$ ubuntu-device-flash query --device=krillin --channel=ubuntu-touch/rc/bq-aquaris.en --list-images

1: description='ubuntu=20150116,device=20150113-2a2e4c5,custom=20150112-494-23-173,version=1'
15: description='ubuntu=20150129,device=20150126-cb82dc1,custom=20150127-526-24-181,version=15'
16: description='ubuntu=20150129,device=20150129-c75dcfb,custom=20150129-528-26-182,version=16'
17: description='ubuntu=20150209,device=20150209-094615f,custom=20150207-538-29-183,version=17'
18: description='ubuntu=20150211.1,device=20150211-74c2df2,custom=20150207-538-29-183,version=18'
19: description='ubuntu=20150219,device=20150216-fe747ac,custom=20150216-561-29-186,version=19'
20: description='ubuntu=20150219,device=20150225-b67e0b6,custom=20150216-561-29-186,version=20'
21: description='ubuntu=20150312,device=20150310-3201c0a,custom=20150216-561-29-186,version=21'
22: description='ubuntu=20150410.1,device=20150408-4f14058,custom=20150409-665-29-206,version=22'
{% endhighlight %}




Show information about the most recent image in the ubuntu-touch/rc/bq-aquaris.en channel:

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




Show information about an older image (r16) on the `ubuntu-touch/rc/bq-aquaris.en` channel, notice that the `--revision=16` parameter goes *before* the `query` command:



{% highlight text %}
$ ubuntu-device-flash --revision=16 query --device=krillin --channel=ubuntu-touch/rc/bq-aquaris.en --show-image

Device: krillin                                                                                                                                                                                                                                                         
Description: ubuntu=20150129,device=20150129-c75dcfb,custom=20150129-528-26-182,version=16                                                                                                                                                                              
Version: 16                                                                                                                                                                                                                                                             
Channel: ubuntu-touch/rc/bq-aquaris.en                                                                                                                                                                                                                                  
Files:                                                                                                                                                                                                                                                                  
 0 https://system-image.ubuntu.com/pool/ubuntu-d09bf94c3f15660b0e92c6f543f94f0b5e43338a6b7c2b6a028a654ed0d6f1f4.tar.xz 295702312 be447e7ba1ff9aa06a7776dbf94e9663fc226038782fc18c0070bd8969d063d4                                                                       
 1 https://system-image.ubuntu.com/pool/device-0519a8c6648de07a6ca9bc7c5a8fdb3961758b1bb49b06d3b1af9beea59223f4.tar.xz 70565660 5e3d11ca0f6e80b629d8a3e7cbbb2adb79351bdcf0e41435bf8732899a32f01a                                                                        
 2 https://system-image.ubuntu.com/pool/custom-1f876a2e9e6273f4771f7bf3c412059f1e295d9380ea61f077b3f323d8f4ead2.tar.xz 58161292 38aa4ca5e713b533d3a838956aa4664c86661fc1df8ef30cd1fd7f2a22d79bec                                                                        
 3 https://system-image.ubuntu.com/ubuntu-touch/rc/bq-aquaris.en/krillin/version-16.tar.xz 372 0ae91f111d0c01a22aceead5222d710ad291c1537212bd6f215eef52437d8767
{% endhighlight %}



The links can be used to download individual parts of an image, the number after the link is the file size in bytes and the string at the end is the SHA256 hash of the file:



{% highlight text %}
$ wget https://system-image.ubuntu.com/pool/custom-1f876a2e9e6273f4771f7bf3c412059f1e295d9380ea61f077b3f323d8f4ead2.tar.xz

$ sha256sum custom-1f876a2e9e6273f4771f7bf3c412059f1e295d9380ea61f077b3f323d8f4ead2.tar.xz
38aa4ca5e713b533d3a838956aa4664c86661fc1df8ef30cd1fd7f2a22d79bec  custom-1f876a2e9e6273f4771f7bf3c412059f1e295d9380ea61f077b3f323d8f4ead2.tar.xz
{% endhighlight %}



## Flash/Download Ubuntu Touch images


Download and flash an image to an attached device which is in fastboot mode (`--bootstrap`), clear all data after flashing (`--wipe`). If `--revision` (see previous examples) is omitted, the most recent image will be flashed. `--channel` can be used to select a different channel than the default one.

The following example has been adapted for the bq Aquaris E4.5 Ubuntu Edition, which ships with a recovery image that does NOT enable ADB by default, leading to a "Failed to enter Recovery" error message. It is necessary to temporarily use a different recovery image, e.g. [this][barajas-recovery] one, and pass the `--recovery-image` parameter. This is usually not necessary with Nexus devices.


{% highlight text %}
$ ubuntu-device-flash touch --bootstrap --wipe --recovery-image /tmp/recovery.img

2015/05/05 15:04:37 Expecting the device to be in the bootloader... waiting
2015/05/05 15:04:37 Device is |krillin|
2015/05/05 15:04:37 Flashing version 21 from ubuntu-touch/stable channel and server https://system-image.ubuntu.com to device krillin
2015/05/05 15:04:52 Start pushing /home/sturmflut/.cache/ubuntuimages/ubuntu-touch/stable/krillin/version-21.tar.xz to device
2015/05/05 15:04:52 Start pushing /home/sturmflut/.cache/ubuntuimages/pool/ubuntu-d2dfa371c65640e688fd9272b3ede7dbddbfed27f548a0d988c083b1d1c78158.tar.xz to device
2015/05/05 15:04:52 Start pushing /home/sturmflut/.cache/ubuntuimages/pool/device-0142302186687e3e48e6987283f6caf5d471a4160f98aa6a3cb7658f96471297.tar.xz to device
2015/05/05 15:04:52 Start pushing /home/sturmflut/.cache/ubuntuimages/pool/custom-80897f6b2dbcbc12a01735e0fc965a6cc2cdb346f1da05b919406a7790565523.tar.xz to device
2015/05/05 15:04:52 Start pushing /home/sturmflut/.cache/ubuntuimages/gpg/image-master.tar.xz to device
2015/05/05 15:04:52 Start pushing /home/sturmflut/.cache/ubuntuimages/gpg/image-signing.tar.xz to device
2015/05/05 15:04:52 Done pushing /home/sturmflut/.cache/ubuntuimages/ubuntu-touch/stable/krillin/version-21.tar.xz to device
2015/05/05 15:04:52 Done pushing /home/sturmflut/.cache/ubuntuimages/gpg/image-master.tar.xz to device
2015/05/05 15:04:52 Done pushing /home/sturmflut/.cache/ubuntuimages/gpg/image-signing.tar.xz to device
2015/05/05 15:05:15 Done pushing /home/sturmflut/.cache/ubuntuimages/pool/custom-80897f6b2dbcbc12a01735e0fc965a6cc2cdb346f1da05b919406a7790565523.tar.xz to device
2015/05/05 15:05:18 Done pushing /home/sturmflut/.cache/ubuntuimages/pool/device-0142302186687e3e48e6987283f6caf5d471a4160f98aa6a3cb7658f96471297.tar.xz to device
2015/05/05 15:05:58 Done pushing /home/sturmflut/.cache/ubuntuimages/pool/ubuntu-d2dfa371c65640e688fd9272b3ede7dbddbfed27f548a0d988c083b1d1c78158.tar.xz to device
2015/05/05 15:05:58 Created ubuntu_command: /home/sturmflut/.cache/ubuntuimages/ubuntu_commands760854353
2015/05/05 15:05:58 Rebooting into recovery to flash
{% endhighlight %}


Flash a device that is in fastboot mode  (`--bootstrap`) with the most recent image from the default channel, clear all data after flashing (`--wipe`), automatically enable developer mode after flashing (`--developer-mode`), set a PIN (`--password=`). After the device has been flashed, use `phablet-config` to bypass the welcome wizard and edges intro, completing a fully automated device install e.g. for continuous integration.

*NOTE:* This does currently not seem to work on the bq Aquaris E4.5 Ubuntu Edition. When the welcome wizard pops up after flashing, the device is not in developer mode and ADB is not enabled.

{% highlight text %}
$ ubuntu-device-flash touch --bootstrap --developer-mode --password=1234

$ phablet-config edges-intro --disable

$ phablet-config welcome-wizard --disable
{% endhighlight %}


Download image revision 1 for device `flo` (Nexus 7)  and channel `ubuntu-touch/vivid`, do not flash.

{% highlight text %}
$ ubuntu-device-flash --download-only --revision=1 touch --device=flo --channel=ubuntu-touch/vivid

2015/05/05 14:54:55 Device is |flo|
2015/05/05 14:54:55 Flashing version 1 from ubuntu-touch/vivid channel and server https://system-image.ubuntu.com to device flo
3.67 MB / 3.67 MB [===========================================================================================] 100.00 % 7.56 MB/s 
46.53 MB / 46.53 MB [========================================================================================] 100.00 % 14.31 MB/s 
321.82 MB / 321.82 MB [======================================================================================] 100.00 % 16.16 MB/s 
2015/05/05 14:55:15 Downloaded files for version 1, channel ubuntu-touch/vivid, exiting without flashing as requested.
{% endhighlight %}



## Ubuntu Snappy Core mode

`ubuntu-device-flash core` creates Ubuntu Snappy Core images for different devices, look [here][ubuntu-snappy]. This has currently nothing to do with phones, so it is out of the scope of this series.



[ubuntu-touch-install]: https://developer.ubuntu.com/en/start/ubuntu-for-devices/installing-ubuntu-for-devices/

[barajas-recovery]: http://people.canonical.com/~jhm/barajas/recovery.img

[ubuntu-snappy]: https://developer.ubuntu.com/en/snappy/guides/porting/