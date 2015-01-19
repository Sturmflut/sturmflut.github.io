---
layout: post
title:  "Designing a WiFi Analyzer app for Ubuntu Touch"
date:   2015-01-19 13:00:00
categories: Linux Wireless
---

I just finished work on the [Flood It clone][github-floodit] for Ubuntu Touch and can't work on the Network Scanner app until the issue mentioned in my [older post]({% post_url 2015-01-17-unprivileged-icmp-sockets-on-linux %}) is resolved, so I decided to start development on a new app: a clone of the popular WiFi Analyzer for Android. Because it was not immediately clear how to build such an app, I did some research first.

The minimum amount of information needed is:

- A list of all access points currently in range

- Access point MAC address

- SSID, frequency, signal strength

- Available authentication and encryption schemes

Optional, but "nice to have" information is:

- Available data rates (so one can distinguish between 802.11b/g/n/ac networks)

Ideally all this information would be updated as fast as the adapter/driver/kernel can deliver it, but it should be updated at least every one to three seconds, maybe five seconds maximum, otherwise the usefulness of the app will be severely limited.

Now there's a major obstacle on Linux: only someone with root privileges can initiate a scan for WiFi networks. Unprivileged users may access the [Wireless Extension API][wext] (Wext) or talk to the kernel via [netlink][nl80211] (nl80211), or parse the output of the corresponding `iwlist scan`  and `iw dev wlan0 scan dump` commands, but both APIs and commands will only report the results of the last scan.

So we need a service outside the confined app, running as root, which periodically initiates scans for us. We also need to care about caching and timeouts for detected networks: the kernel does its best to return an up-to-date list of detected networks, but the timeout seems to be at least 30 seconds, which is way too long for a decent WiFi Analzyer.

In a perfect world, the network management service at the heart of the operating system would do all the work for us and expose the information to our unconfined app via some kind of API. So wait, who does all the network management on Ubuntu Touch? NetworkManager, [and it exports all the information we need via D-Bus][network-manager-dbus]! NetworkManager will probably be replaced by [systemd-networkd] at some point, but the systemd folks plan to expose a D-Bus API as well, and supporting both would probably not be that much work.

Sadly there are two problems:

- As [jdstrand] pointed out on #ubuntu-touch, confined apps are not allowed to talk to NetworkManager via D-Bus. They currently can, but it is considered a bug, because this way any app can mess with the network configuration.

- The default scan interval used by NetworkManager is extremely long (seems to be 15 seconds), and if the service is busy with other things, e.g. connecting to an access point, it will defer scans. This is unsuitable for a WiFi Analyzer.

So what now? The proper way to communicate with the network management subsystem is the Ubuntu [connectivity-api]. It is currently limited to exactly two API calls (see [the source][connectivity-api-source]):

{% highlight C++ %}
bool NetworkingStatus::online() const
bool NetworkingStatus::limitedBandwith() const
{% endhighlight %}

It doesn't look like a WiFi Analyzer app could be built until somebody dramatically extends the connectivity-api and decreases NetworkManager's scan interval.

If you have any ideas, you can usually find me on the FreeNode IRC.


[github-floodit]: https://github.com/Sturmflut/ubuntu-touch-floodit
[connectivity-api]: https://launchpad.net/connectivity-api
[wext]: http://wireless.kernel.org/en/developers/Documentation/Wireless-Extensions
[nl80211]: http://wireless.kernel.org/en/developers/Documentation/nl80211
[network-manager-dbus]: http://cgit.freedesktop.org/NetworkManager/NetworkManager/tree/introspection/nm-access-point.xml
[systemd-networkd]: http://www.freedesktop.org/software/systemd/man/systemd-networkd.service.html
[jdstrand]: https://launchpad.net/~jdstrand
[connectivity-api]: https://launchpad.net/connectivity-api
[connectivity-api-source]: https://bazaar.launchpad.net/~unity-api-team/connectivity-api/trunk.15.04/view/head:/src/qt/qml/Ubuntu/Connectivity/networking-status.cpp