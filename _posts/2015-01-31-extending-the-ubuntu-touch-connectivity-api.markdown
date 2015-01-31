---
layout: post
title:  "Extending the Ubuntu Touch connectivity-api"
date:   2015-01-31 23:00:00
categories: Ubuntu Touch
---

After my [post about designing a WiFi scanner for Ubuntu Touch]({% post_url 2015-01-19-designing-a-wifi-analyzer-app-for-ubuntu-touch %}) I went to #ubuntu-app-devel and asked around what would be the correct and supported way to get information about WiFi networks. Turns out that it really is the connectivity-api, and as Wellark pointed out it is not only designed to be responsible for WiFi networks, but for pretty much anything that goes through an antenna: WiFi, mobile networks, Bluetooth.

The connectivity-api is currently extremely limited and the design is not final, which gives us the opportunity to design it from scratch. I created a series of articles detailing my requirements from the view of scanner applications:

- [Extending the Ubuntu Touch connectivity-api for WiFi scanning]({% post_url 2015-01-30-extending-the-ubuntu-touch-connectivity-api-for-wifi-scanning %})
- [Extending the Ubuntu Touch connectivity-api for Bluetooth scanning]({% post_url 2015-01-31-extending-the-ubuntu-touch-connectivity-api-for-bluetooth-scanning %})
- [Extending the Ubuntu Touch connectivity-api for mobile network scanning]({% post_url 2015-01-31-extending-the-ubuntu-touch-connectivity-api-for-mobile-network-scanning %})

I filed a series of bugs against the connectivity-api, see the coresponding article for the link to the Launchpad bugtracker.