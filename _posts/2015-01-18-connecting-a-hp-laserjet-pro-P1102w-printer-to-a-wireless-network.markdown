---
layout: post
title:  "Connecting a HP LaserJet Pro P1102w printer to a wireless network"
date:   2015-01-18 19:00:00
categories: Printer Wireless
---

***TL;DR if you intend to buy a HP LaserJet Pro P1102w and want to connect it to a contemporary wireless network, don't buy it. It will probably not work with your existing setup.***

The HP LaserJet Pro P1102w looks like a pretty decent printer: compact build, 600 x 600 dpi monochrome laser unit, up to 19 pages per minute, built-in WiFi connection, cheap toners, competitive price. And in contrast to other cheap WiFi-enabled printers on the market, HP offers proper Linux drivers and Linux utilities which are available in all major Linux distributions.

So I picked up a unit at a local electronics retailer and tried to integrate it into my home network. At the beginning everything looked fine: the printer doesn't have a front panel, so it has to be connected via USB for initial WiFi configuration. The HP wizard popped up instantly, discovered the printer and scanned for available wireless networks. I selected my network, put in the passphrase and waited. After a couple of minutes an error message popped up, telling me that the printer didn't show up on the network as expected. I found a longstanding [thread][hp-forums] on the HP forums about this issue, and after an hour of debugging I think I've found most of the problems.

The built-in WiFi module and the printer firmware seem to not implement many WiFi aspects correctly:

- Only channels 6 and 11 are officially supported. Other channels may work, but there's no guarantee.

- Only 802.11b and 802.11g are supported. This should be okay, as WiFi is backwards compatible, but the P1102w doesn't seem to like it if the access point also simultaneously offers 802.11n. It ends up being a major problem, since you really want your other devices to communicate at 802.11n speeds.

- Only WEP and WPA-PSK seem to be supported. This is a major problem, since most customer routers/access points default to  WPA2 and you don't want to switch back to WPA-PSK for security reasons. The P1102w also doesn't seem to like it if the access point simultaneously offers WPA and WPA2.

- The P1102w doesn't seem to like SSIDs which contain spaces. Dashes seem to be okay.

- WPS doesn't seem to work at all.

- In some cases the passphrase set via the GUI tools may not be equal to the passphrase that the device ends up using, especially when using WEP and if the passphrase contains special characters (spaces etc.)

- In some cases the GUI shows the new configuration, but the printer is still probing the old network. In this case the printer has to be manually reset by starting it with both buttons pressed.

Luckily one of my access points supports the creation of separate guest networks, so I created a special wireless network with the following settings: 802.11b+g only, channel 11, SSID "storm-printer", WPA-PSK with a password that only contains lowercase letters. I created the necessary firewall rules so that the hosts on my network can reach the printer. The printer now reliably connects, and I can successfully print from my Ubuntu 14.10 and 15.04 installations.

I don't expect the average user to juggle multiple wireless networks just to get a printer online, so I can't recommend this device.


[hp-forums]: http://h30434.www3.hp.com/t5/Printer-Networking-and-Wireless/P1102W-not-connecting-wirelessly/td-p/261530
