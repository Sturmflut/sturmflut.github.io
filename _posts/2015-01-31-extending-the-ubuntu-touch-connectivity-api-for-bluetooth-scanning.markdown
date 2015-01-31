---
layout: post
title:  "Extending the Ubuntu Touch connectivity-api for Bluetooth scanning"
date:   2015-01-31 20:00:00
categories: Ubuntu Touch
---

One of my Ubuntu Touch app ideas is a Bluetooth Scanner. Bluetooth management functionality will have to be implemented anyways, and since Qt Bluetooth is not part of Ubuntu Touch images, it is time for yet another extension of the [connectivity-api].


### Technology ###

Bluetooth is a short-range, low-speed and low-power communication technology operating in the same 2.4 GHz band as most WiFi standards. There are no access points, but communication is mostly point-to-point. Devices can only communicate with each other after a "pairing" process. No problem here, everybody knows this much.

From the view of a scanning application, there are several problems:

- "Bluetooth Classic/High-Speed" and "Bluetooth Low Energy" (also called "Bluetooth Smart") are incompatible. A device may support both or either one, so if the device the scanner is running on offers both, scans for both technologies have to be initiated. I am not sure if all dual-mode devices allow both modes to be active at the same time, since the manufacturer may decide to share parts of the silicon and the antenna(s).

- Bluetooth Classic devices do not announce their presence regularly, the host adapter has to actively scan its neighborhood. Depending on the protocol/hardware version it takes from 12 up to 30 seconds to discover all devices in range, and if devices are found, they have to be contacted individually to gather additional information.

- Bluetooth Low Energy devices actively advertise their presence in regular intervals (20 Milliseconds up to ten seconds), but there are three different advertisement channels and the transmitting device uses them sequentiall. The scanner device has to listen for advertisements and actively query detected devices for additional information. When a Bluetooth Low Engery device is connected to another device, it also stops advertising! Depending on implementations, advertisement intervals and noise it may therefore take minutes until every advertisement packet has reached the scanner, if devices are advertising at all.

- There is no need for the operating system to regularly scan for Bluetooth Classic devices or to actively listen for Bluetooth Low Energy devices all the time, so the scanner has to trigger the correct operation for each technology, cache the results, remove stale entries etc.

A Bluetooth scanner will therefore not be able to cope well with highly dynamic environments, since it may take several minutes until all devices are discovered. And depending on the device it may not be possible to scan both technologies at the same time.


### Device types and needed information ###

The purpose of a Bluetooth scanner is to find *all* devices in range and gather as much additional information as possible, so we have to keep in mind which device types exist and which information we care about.

- Bluetooth Classic: At least the device MAC address and the contents of the "Class of Device" field are needed. If the device has a "pretty name" set, we want to know it, and if the adapter reports the signal strength with which the information was received this is important as well. We also want to query devices for the exact list of services they offer.

- Bluetooth Low Energy: Advertisements contain up to 31 bytes of arbitrary information, and the scanning device may actively query for up to 31 additional bytes. This information is important because it contains device names, UUIDs etc. The list of services offered can be retrieved using the Generic Attribute Profile (GATT), so the API should offer the necessary calls.


### API calls and features ###

I propose the following API functions:

- `startScan()` initiates a scan using the given technology. For Bluetooth Classic this would trigger an active scan, while for Bluetooth Low Energy it would set the device to continuous listening mode. Since it is unknown if and when the scan is finished, the call should return immediately and information is returned via callbacks. Depending on the hardware it may be possible to start scans for the different technologies at the same time.

- `stopScan()` stops an ongoing scan.

- `queryServices()` takes a device address and actively queries for a list of all offered services. The specification mentions quite short timeouts for many message types, so I am not sure if this call should block or use a callback.


### Security and Privacy ###

A hostile app may calculate the location of a user from the list of iBeacons in range, and because of the short-range nature of Bluetooth this location will be quite accurate. I therefore propose that the first call to `startScan()` triggers a system popup informing the user about possible privacy implications and asking for permission.


The corresponding Launchpad bug report is [1416731][launchpad-bug-connectivity-api-bluetooth]. As always you can find me on the FreeNode IRC.


[connectivity-api]: https://launchpad.net/connectivity-api
[launchpad-bug-connectivity-api-bluetooth]: https://bugs.launchpad.net/connectivity-api/+bug/1416731