---
layout: post
title:  "Ubuntu Touch app wishlist"
date:   2015-02-15 22:00:00
categories: Ubuntu Touch
---


Basic functionality is there, the most important APIs have been implemented, the first phone has been released and in about four weeks the first lucky users will get their phones. Time to fill the app store!

Over the last couple of days I compiled the following wishlist. An entry on this list does not necessarily mean that I use the app myself, it just means that I have identified a need for it.

If you are looking for an app idea to implement, or are looking for a developer who you can nag into porting his app to Ubuntu Touch, please consider this list.




## Clones of classic games

- 4 in a row/Connect Four/etc.

- Backgammon

- Battleships

- [Bindoku][bindoku] (Binary Sudoku)

- Breakout

- Card games

- Checkers and its variants (e.g. Chinese Checkers)

- [Gorillas][gorillas]

- Mastermind

- Math games (basic algebra etc.)

- Minesweeper

- Monopoly (maybe with online multiplayer support?)

- Risk

- SameGame

- Snake

- Tic-Tac-Toe

- Trivial Pursuit. There was a digital version of the game once, with downloadable trivia question packages, maybe the file format could be reverse-engineered and the files loaded if the user provides them?




## Clones of popular apps

I think we need clones of these because Ubuntu Touch is too different from other platforms for a direct port.

- AdBlock/AdFree

- Audio/Video format converter

- Backup. This is extremely important, phones store a lot of data and most users will get a new phone within less than three years, so backup and restore should be as painless as possible. It's a horror on Android, let's do better.

- A Camera app on par with the Google Camera App. I use the Panorama and Photo Sphere function quite often.

- Native clients for all major cloud file sync services (Syncthing, ownCloud, DropBox, MEGA etc.)

- DLNA server to stream audio/video to the local network.

- FTP/SCP/SFTP file transfer client, maybe built into the file manager? And with Content Hub support?

- [Folder size][foldersize]

- Desktop integration like AirDroid. Transfer files between desktop and phone, show phone notifications on the desktop, write SMS from the desktop etc.

- [TCP/IP network scanner]({% post_url 2015-01-17-unprivileged-icmp-sockets-on-linux %}) (as soon as the situation regarding unprivileged ICMP sockets has been resolved)

- [WiFi scanner]({% post_url 2015-01-30-extending-the-ubuntu-touch-connectivity-api-for-wifi-scanning %}) (as soon as the connectivity-api exposes the necessary functionality)

- Simple video editor

- Task automation like [Tasker][tasker]

- Web Radio client





## Clones/Ports of useful single-purpose database apps

These apps could probably be ported, but they are so simple that they can be easily cloned.

- [Dangerous goods][gefahrguthelfer]

- [License plates][kennzeichen]

- Country and State Capitals





## Ports of popular, commercial games

I think some of them already have an HTML5 version, so they could possibly be ported as fast as Cut the Rope.

- Angry Birds

- Bio. Inc/Plague Inc./Biotix etc.

- Candy Crush Saga

- Clash of Kings/Clash of Clans/Goodgame Empire/etc.

- Fruit Ninja

- Heroes Charge

- Hill Climb Racing

- Monument Valley

- Multiplayer online quizzes

- Plants vs. Zombies (we have [Machines vs. Machines][machines-vs-machines] though!)

- Pou/Tamagotchi/etc.

- Subway Surfer

- Temple Run

- The Sims

- Worms

- etc.

Just look at the Android App Store Charts.





## Ports of popular open source apps

The source code for the following apps can mostly be found somewhere on the Internet, so it might be possible to port something.

- [Aard Dictionary][aard]. Aard allows you to read offline versions of many dictionaries and databases, e.g. Wikipedia, Wikivoyage, Wikiquote etc. I use it extensively when traveling. There are several open-source implementations of the libraries and frontends for different platforms, e.g. Android and Desktop Linux.

- Firefox

- [OsmAnd][osmand] (navigation with offline maps support). I use it extensively, not just abroad.

- Password Safes. I don't use them and don't like them, but other people do.

- Remote frontends for Torrent clients (Deluge etc.)

- Scuba Diving logger and gas calculator. There are a lot of divers out there, including Torvalds himself.

- Tor

- Vector graphics editor like SketchBook Express.

- [VideoLAN Client (VLC)][videolan]





## Ports of popular open source apps

The source code for the following apps is not available, and they often rely on proprietary APIs, so we probably have to nag the authors.

- Booking.com, Airbnb and friends. There are webapps, but native apps come with useful features like offline support.

- Carsharing apps. A webapp may be enough, but I am not sure as I don't use Carsharing.

- E-Learning for Languages and other areas.

- Fitness tracker apps like FitBit, FitApp, Runtastic etc.

- Flipboard

- Google Hangouts or a replacement. WebRTC might be an option.

- Instagram/Snapchat/Flickr/Slingshot/etc.

- Language Translator with offline support.

- LinkedIn. There is a webapp, but it lacks platform integration.

- Native Facebook and Facebook Messenger apps. I don't like Facebook, but people are going to want it, and the webapp is powerful, but lacks platform integration.

- An Office Suite, preferably LibreOffice.

- Online Banking for multiple accounts.

- Public transportation apps like [DB Navigator][db-navigator] or [Handyticket.de][handyticket]. [fahrplan][fahrplan] already implements many things, but you can't e.g. buy tickets with it, which is one of my major use cases for a smartphone. At least Deutsche Bahn was well-known for also releasing their apps for more "exotic" platforms like Palm handhelds or Symbian devices, and lots of functionality in the Android app is just implemented as a web views, so there is a slight possibility for a somewhat native UT app.

- Shazam or any other song recognition service.

- Skobbler. We will never get a native, full-featured Google Maps app anyway, and Skobbler might be a worthy replacement.

- Skype or Viber or LINE or Tango or Wire or a replacement, but the replacement will have to be *very* good if it wants to successfully compete against Skype.

- [Speedtest.net][speedtest]

- TripAdvisor: There is a webapp, but the native app can download content for offline use.

- WhatsApp. I don't like it, and most smartphones have the power to run both WhatsApp and Telegram at the same time, so you can probably talk most of your friends into installing Telegram, but when Ubuntu Touch devices become generally available the lack of a WhatsApp client will become problematic. And no, WhatsApp Web is not a solution.

- WeChat for chinese users.

- Xing. There is a webapp, but it lacks platform integration.

- Zedge





[tasker]: https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm&hl=de

[bindoku]: https://play.google.com/store/apps/details?id=de.lochmann.bindoku

[gorillas]: http://en.wikipedia.org/wiki/Gorillas_%28video_game%29

[foldersize]: https://play.google.com/store/apps/details?id=com.foldersize&hl=de

[gefahrguthelfer]: https://play.google.com/store/apps/details?id=at.knorre.dangerousgoods&hl=de

[kennzeichen]: https://play.google.com/store/apps/details?id=flaotec.KFZ_Kennzeichen&hl=de

[machines-vs-machines]: https://appstore.bhdouglass.com/app/com.ubuntu.developer.mzanetti.machines-vs-machines

[aard]: http://aarddict.org/

[db-navigator]: http://www.bahn.de/p/view/buchung/mobil/db-navigator.shtml

[handyticket]: https://www.handyticket.de/

[fahrplan]: https://appstore.bhdouglass.com/app/com.ubuntu.developer.mzanetti.fahrplan2

[osmand]: http://osmand.net/

[speedtest]: http://www.speedtest.net/

[videolan]: http://www.videolan.org/vlc/