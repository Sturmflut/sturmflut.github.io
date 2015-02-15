---
layout: post
title:  "Using the T-Mobile Speedstick LTE II on Ubuntu 14.10"
date:   2015-02-15 16:30:00
categories: Linux Ubuntu
---

Telekom Germany decided to give everyone who cares [a free SIM with 2 x 5GB of LTE traffic at maximum speed][data-comfort-free]. I am using it with the [T-Mobile Speedstick LTE II][amazon-speedstick] on Ubuntu 14.10.

{% highlight bash %}
$ lsusb
Bus 003 Device 006: ID 1bbb:011e T & A Mobile Phones
{% endhighlight %}

The device is recognized by usb_modeswitch and NetworkManager/ModemManager as shipped with Ubuntu 14.10, but usb_modeswitch would fail about 19 out of 20 tries, especially when connecting the stick to an USB 3.0 port. I changed the following value in `/etc/usb_modeswitch.conf`

{% highlight bash %}
SetStorageDelay=5
{% endhighlight %}

and rebooted the installation. Now usb_modeswitch seems to be able to reliably switch the device.


[data-comfort-free]: https://www.t-mobile.de/data-comfort-free/0,26298,28534-_,00.html
[amazon-speedstick]: http://www.amazon.de/T-Mobile-Speedstick-LTE-II-Surfstick-Wei%C3%9F/dp/B00AEGSBJC