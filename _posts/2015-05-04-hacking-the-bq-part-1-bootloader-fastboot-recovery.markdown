---
layout: post
title:  "Hacking the bq, Part 1: Bootloader, Fastboot and Recovery"
date:   2015-05-04 21:00:00
categories: Ubuntu bq
---

**UPDATE 11.05.2015: Found a third way of getting into Fastboot mode.**


## Bootloader

To access the Bootloader Menu using the device alone, do the following:

* Unplug the USB cable (if necessary)
* Turn off the phone
* Press the POWER and VOLUME UP buttons simultaneously until the screen turns on

You can use the VOLUME UP button to cycle through the options and the VOLUME DOWN button to execute the currently selected option.


## Fastboot

Method 1: Boot into the Bootloader and execute the "FASTBOOT" entry.

Method 2: To access the Bootloader Menu using an attached computer, while the device is powered on:

* Make sure the phone is in developer mode
* Make sure `adb devices` lists the device
* Run `adb reboot bootloader`
* Make sure `fastboot devices` lists the device

Method 3: Turn the device off and then press all three keys (POWER, VOLUME UP and VOLUME DOWN) until the display turns on (about three seconds).

You can now use the `fastboot` command to work with the device.

To exit FASTBOOT mode, either press the POWER and VOLUME UP buttons until the phone turns off, or execute `fastboot reboot` if the phone still responds via USB.


## Recovery

Method 1: Boot into the Bootloader and execute the "RECOVERY" entry. Once the Ubuntu circle shows up, press the VOLUME UP button.

Method 2: To access the Bootloader Menu using an attached computer, while the device is powered on:

* Make sure the phone is in developer mode
* Make sure `adb devices` lists the device
* Run `adb reboot recovery`
* Once the Ubuntu circle shows up, press the VOLUME UP button

You are now in the Recovery menu. It offers the following three options:

* reboot system now
* wipe data/factory reset
* wipe cache partition

You can use the VOLUME DOWN button to cycle through the options and the POWER button to execute the currently selected option.
