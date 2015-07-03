---
layout: post
title:  "Mediatek details: Little Kernel"
date:   2015-07-05 20:00:00
categories: MediaTek
---


*NOTE: This is a continuation of the [previous article]({% post_url 2015-07-04-mediatek-details-partitions-and-preloader %}) in the series. The information was obtained from various sources and through reverse engineering, don't take it as a reference!*


The Preloader has done its work, but you might have noticed that some crucial parts of the picture are still missing.

1. Who cares about the charging animation when the phone is off and the USB charger is attached? The Preloader doesn't.

2. Who shows the Boot Menu and then actually executes the correct boot mode, e.g. Recovery instead of normal boot? The Preloader passes on some boot arguments, but it always loads and executes the same flash partition at the end.

3. Who shows the boot logo during normal boot? The Preloader doesn't.

4. Who implements all those cool Android features (Boot menu, Recovery, Fastboot)?  The Preloader certainly doesn't.

There must be some crucial piece missing between the Preloader and the actual Operating System kernel. On MediaTek SoCs, this component is called "Little Kernel" (LK). It is a kind of second stage bootloader, because the Preloader has to be small enough to fit into the On-chip SRAM when loaded by the Boot ROM.

Sadly I could not find the full source code for LK, the core parts are missing. But we can have a look at the stuff that's there, and we can infere some things.

Tho following sections do not necessarily correspond to sections in the code.




## Platform Init

Yet again the first step will be to bring up all necessary hardware, most importantly the flash storage, power management circuits, the GPIO pins and the display. The interesting thing here is that the Little Kernel comes with a minimal display driver for the used platform in `mediatek/platform/${platform}/lk/mt_disp_drv.c`, in case of the MT6582 it's a small driver for the ARM Mali GPU that just enables the display, allocates a full-screen framebuffer in RGB565 format and hard-wires the GPU to display it.

The first step after hardware bring-up is to load the logo from the `LOGO` partition. Since there is no color space conversion available at the driver level, the logo has to be stored in "raw" format. The `mediatek/custom/common/lk/logo/tool/bmp_to_raw` tool can be used to convert BMP images. Note that the logo is only loaded in this step, but not yet displayed.



## Chose the boot mode

Execution of LK may start for various reasons: The device is off and the charger connected, the device was rebooted, the device is powered on normally etc. All of these modes require different code paths, and the manufacturer may also want to give the user the possibility to modify the boot mode by pressing the hardware keys. As always much of the functionality can be enabled or disabled by the manufacturer, and potential key bindings changed.

Let's remember the available boot modes:


{% highlight text %}
typedef enum
{
    NORMAL_BOOT = 0,
    META_BOOT = 1,
    RECOVERY_BOOT = 2,
    SW_REBOOT = 3,
    FACTORY_BOOT = 4,
    ADVMETA_BOOT = 5,
    ATE_FACTORY_BOOT = 6,
    ALARM_BOOT = 7,
#if defined (MTK_KERNEL_POWER_OFF_CHARGING)
    KERNEL_POWER_OFF_CHARGING_BOOT = 8,
    LOW_POWER_OFF_CHARGING_BOOT = 9,
#endif
    FASTBOOT = 99,
    DOWNLOAD_BOOT = 100,
    UNKNOWN_BOOT
} BOOTMODE;
{% endhighlight %}



The following possibilities to modify the boot mode during LK initialisation exist:

* The Operating System can set two special bits in the memory of the real-time clock and reboot the device. If set, one of those bits will switch the LK into `FASTBOOT` mode, and the other into `RECOVERY_BOOT`. This is how the `adb reboot [bootloader|recovery]` command works.

* Various keys and key combinations will switch to `RECOVERY_BOOT`, `FACTORY_BOOT` or the Boot Menu. The manufacturer can define these keys for every device, look [here]({% post_url 2015-05-04-hacking-the-bq-part-1-bootloader-fastboot-recovery %}) and [here]({% post_url 2015-05-04-hacking-the-bq-part-2-factory-mode %}) for examples for the bq Aquaris E4.5 Ubuntu Edition.



## Boot Menu

If triggered by the hardware key during startup, LK will switch to a text mode emulation and display the following Boot Menu:


{% highlight text %}
Select Boot Mode:
[VOLUME_UP to select.  VOLUME_DOWN is OK.]

[Recovery    Mode]
[Fastboot    Mode]   
[Normal      Boot]   
[Normal      Boot +ftrace]
[Normal      slub debug off]
{% endhighlight %}

Option 1 will select `RECOVERY_BOOT`, option 2 `FASTBOOT` and all other options `NORMAL_BOOT`. The last two options will add the flags `trace_buf_size=11m boot_time_ftrace` or `slub_debug=-` to the kernel commandline.

The last two options are usually not enabled on production devices.



## Further initialisation

Like the Preloader, LK uses the SecLib binary blob.

At this step some more hardware, like the battery controller and the RTC, is initialised, and almost all boot modes will have LK show the logo now.

The `KERNEL_POWER_OFF_CHARGING_BOOT` and `LOW_POWER_OFF_CHARGING_BOOT` modes for charging while the phone is powered off use the notification LED to indicate charging status. The code actually distinguishes between three charging levels: "low", "medium" and "full". But for some reason both "low" and "medium" are hardwired to the same color: Red.




## Handle the boot mode


From now on most information has to be inferred from the remaining code and the context, because the `main` function is missing.


* `NORMAL_BOOT`: Load an Android boot image from the `BOOTIMG` partition, verify it and boot the image.

* `META_BOOT`: This is a special "Mobile Engineering Testing Architecture" mode for BaseBand testing during device development. It gets set by the Preloader if requested by the Flash Tool. Looks like it involves booting into a special test mode.

* `RECOVERY_BOOT`: Load the Recovery image from the `RECOVERY` partition, verify it and boot the image.

* `SW_REBOOT`: Is not used in the present code.

* `FACTORY_BOOT`: Like `NORMAL_BOOT`, but LK passes the value "4" to the next stage, which ends up in the `/sys/class/BOOT/BOOT/boot/boot_mode` file of the Linux kernel at the very end, is evaluated by a modified `init` in the initrd and gets `/system/bin/factory` started instead of Ubuntu.

* `ADVMETA_BOOT`: Most likely an advanced version of `META_BOOT`. It gets set by the Preloader if requested by the Flash Tool.

* `ATE_FACTORY_BOOT`: This is a second factory mode that can be enabled during the Preloader stage and is intended for automated calibration of the BaseBand part on the production line. LK passes the value "6" to the next stage, which again ends up in the `/sys/class/BOOT/BOOT/boot/boot_mode` file of the Linux kernel at the very end. But the modified MTK init doesn't seem to distinguish between both Factory modes?

* `ALARM_BOOT`: Like a `NORMAL_BOOT`, but probably sets the `boot_reason` kernel command line parameter to some unknown value.

* `KERNEL_POWER_OFF_CHARGING_BOOT` and `LOW_POWER_OFF_CHARGING_BOOT`: Show the charging animation for a while and then put the display to sleep until the power button is pressed.

* `FASTBOOT`: Power on the USB port and start the Fastboot interpreter, which is part of LK.

* `DOWNLOAD_BOOT`: This is set by the Preloader if the Preloader was put into Download mode. I don't know why LK cares about it, because LK has no Download mode, it has Fastboot.


This is about all I can say about Little Kernel. If you know better and/or something has changed, please find me on Launchpad.net or the Freenode IRC and do get in contact!




## Resources

- [Thunder-Kernel][thunderkernel]

- [Introduction of MTK Tools][mtk-tools]




[thunderkernel]: https://github.com/suribi/Thunder-Kernel

[mtk-tools]: http://wenku.baidu.com/view/d44d5cd9ad51f01dc281f129.html