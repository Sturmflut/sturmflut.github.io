---
layout: post
title:  "Extending the Ubuntu Touch connectivity-api for WiFi scanning"
date:   2015-01-30 18:00:00
categories: Ubuntu Touch
---

One of the Ubuntu Touch apps I'm currently developing is a [WiFi scanner]({% post_url 2015-01-19-designing-a-wifi-analyzer-app-for-ubuntu-touch %}). I think a platform should support this kind of application because many users need the functionality to debug and optimize their setups, and a mobile device has the perfect form factor if you're walking around trying to find out how strong the signal is at a given location and/or which frequency band and channel to use. Some WiFi scanners hit the 50 million downloads mark in the Android App Store. I think a WiFi scanner written in Qt/QML and using the connectivity-api is not only useful on the phone, but also the desktop, since there is currently no user-friendly Linux GUI app available for this purpose.

Exposing information about detected WiFi networks through the connectivity-api would not only benefit WiFi scanner apps, but also enable location-based services.

### Needed information ###

As stated in my original article, the minimum amount of information needed is:

- A list of all broadcasting stations (BSS/IBSS) in range

- Broadcast station MAC address

- SSID (may be emtpy)

- Center frequency or channel (the one can be converted into the other)

- Signal strength

- Available authentication and encryption schemes

- Timestamp when the station was last seen

This is basically the same information as encapsulated in an [android.net.wifi.ScanResult][android-scanresult] object on Android or exposed via D-Bus by NetworkManager.

Optional, but "nice to have" information would be:

- The contents of the "Capability Info" field of the beacon frame.

- Supported data rates, so one can distinguish between 802.11b/g/n/ac networks. This information is encoded in the "Rates" and "Supported MCS Set" fields of the beacon frame.

- The contents of the "HT Capability Info" field (if present) of the beacon frame.

This optional information is already recorded by the kernel and is e.g. decoded/displayed by a call to `iw dev wlan0 scan`.


### API calls and features ###

Since the network management service has to monitor its surroundings anyways, e.g. to detect and connect to an already known network or to switch to a better access point during movement, the information should be readily available. A single `scanResults()` call returning a list of broadcast station objects should therefore already be enough.

I propose the following additional API features:

- A method to register a callback function which is called after every successful scan. This frees the application from polling the API, may save power and the app can process new information instantly.

- A method to set the scan interval within a pre-defined range. The network management service may decide to adapt its scan interval to the current situation to save power, e.g. a longer interval when not currently connected and a short one when connected. A scanner app may require a short, constant interval.

The corresponding Launchpad bug report is [1415098][ubuntu-bug-connectivity-api-wifi-scanning]. As always you can find me on the FreeNode IRC.


[android-scanresult]: https://developer.android.com/reference/android/net/wifi/ScanResult.html
[ubuntu-bug-connectivity-api-wifi-scanning]: https://bugs.launchpad.net/connectivity-api/+bug/1415098
