---
layout: post
title:  "Hacking the bq, Part 4: Building and booting a kernel"
date:   2015-07-02 02:00:00
categories: Ubuntu bq
---

*NOTE: Thanks to Ondrej Kubik and Oliver Grawert from Canonical for their input.*


Sometimes you want to run your own kernel on the device because the production kernels don't have all the interesting options enabled. Or you want to change the kernel commandline used by the bootloader.

The following instructions have been tested with the bq Aquaris E4.5 Ubuntu Edition.


## Installing the prerequisites


{% highlight text %}
$ sudo apt-get install gcc-arm-linux-androideabi abootimg
{% endhighlight %}

Without these installed, the build will silently fail.



## Building the kernel

Check out the `aquaris-E4.5-ubuntu-master` branch:


{% highlight text %}
$ git clone https://github.com/bq/aquaris-E4.5.git -b aquaris-E4.5-ubuntu-master aquaris-E4.5-ubuntu-master

$ cd aquaris-E4.5-ubuntu-master
{% endhighlight %}


Change the kernel configuration (if necessary):


{% highlight text %}
$ vim mediatek/config/krillin/autoconfig/kconfig/project
{% endhighlight %}


Build the kernel:


{% highlight text %}
$ ./makeMtk -t krillin n k
{% endhighlight %}


Build the Android boot image:


{% highlight text %}
$ cd testboot/

$ ./mkbootimg.sh
{% endhighlight %}


You should end up with a `boot.img` file.



## Booting the new kernel once


Boot the phone into [Fastboot]({% post_url 2015-05-04-hacking-the-bq-part-1-bootloader-fastboot-recovery %}) mode and then run the following command on the host:


{% highlight text %}
$ fastboot boot boot.img
{% endhighlight %}


The image will be downloaded, the phone will take a bit to process it and then run the modified kernel until the next reboot.



## Permanently booting the new kernel


Boot the phone into [Fastboot]({% post_url 2015-05-04-hacking-the-bq-part-1-bootloader-fastboot-recovery %}) mode and then run the following commands on the host:


{% highlight text %}
$ fastboot flash boot boot.img

$ fastboot reboot
{% endhighlight %}


The new kernel has now been permanently flashed to the device and will be booted until it is overwritten by the user or an OTA update.



## Changing the kernel command line


An Android boot image consists of the kernel image, an initrd and a `bootimg.cfg` parameter file for the bootloader. The `bootimg.cfg` supports a `cmdline` parameter, whose value will be appended to the default kernel command line.

The easiest way to change this is to use `abootimg` and change this parameter in-place:


{% highlight text %}
$ abootimg -u boot.img -c "cmdline=debug"
{% endhighlight %}


Now boot the image as shown above.


If you know better and/or something has changed, please find me on Launchpad.net and do get in contact!













