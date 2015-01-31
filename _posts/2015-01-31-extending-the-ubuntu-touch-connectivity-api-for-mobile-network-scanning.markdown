---
layout: post
title:  "Extending the Ubuntu Touch connectivity-api for mobile network scanning"
date:   2015-01-31 22:00:00
categories: Ubuntu Touch
---

Mobile networks are scary places. Inadequate encryption, fake base stations installed by criminals, IMSI catchers used by law enforcement agencies and secret services, apps tracking your location using cell towers. The initial customer base for Ubuntu Touch phones will probably be developers and users unhappy with Android and iOS. I think exposing as much information as possible about mobile networks opens the door for interesting applications, e.g. IMSI catcher detectors, free cell tower location databases, location-based services etc.


### Technology ###

There are currently at least four different mobile network technologies available around the world:

- GSM

- LTE

- CDMA

- WCDMA (UMTS)

A device may support one or more technologies at the same time, and the currently available status may therefore change over time. For example an LTE device may temporarily roam down to UMTS or even GSM.



### Needed information ###

The available information differs between technologies, and new technologies will be introduced during the lifetime of the API, so the design should be extensible.

- GSM: A base station is identified by its Cell-ID (CID), Location Area Code (LAC), Mobile Country Code (MCC) and Mobile Network Code (MNC). The last digit of the CID may encode the antenna number if the same base station has more than one. Signal strength may be reported in ASU, dBm or "levels". A Timing Advance value is used to coarsely describe the distance between the mobile device and the currently used base station.

- LTE: A base station is identified by its Cell-ID (CID), Mobile Country Code (MCC), Mobile Network Code (MNC), Physical Cell ID and Tracking Area code. Signal strength may be reported in ASU, dBm or "levels". Timing Advance values are available for every base station of the same operator within range, not only for the currently used one.

- CDMA: A base station is identified by its Base Station ID, Latitude, Longitude, Network ID and System ID. Signal strength may be reported in ASU, dBm, Ec/Io, "levels" or SNR. EVDO for faster data transfer may be available.

- WCDMA (UMTS): A base station is identified by its Cell-ID (CID), Mobile Country Code (MCC), Mobile Network Code (MNC) and Primary Scrambling Code. Signal strength may be reported in ASU, dBm or "levels".


### API calls and features ###

The Baseband chip has to monitor the whole network at all times anyway, so the required information should be readily available. A single call a la `networkCells()` should therefore suffice. The rest of the API is heavily dependent on which methods the Baseband offers and which data it exposes.


### Security and Privacy ###

A hostile app may calculate the location of a user from the list of cell towers in range. Because of the long-range nature of mobile networks this location will often not be accurate, but especially in cities it may be accurate enough. I therefore propose that the first call to `networkCells()` triggers a system popup informing the user about possible privacy implications and asking for permission.



The corresponding Launchpad bug report is [1416741][ubuntu-bug-connectivity-api-mobile-network-scanning]. As always you can find me on the FreeNode IRC.


[ubuntu-bug-connectivity-api-mobile-network-scanning]: https://bugs.launchpad.net/connectivity-api/+bug/1416741