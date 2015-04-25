---
layout: post
title:  "Ubuntu Touch app wishlist and status (April 2015)"
date:   2015-04-24 22:00:00
categories: Ubuntu Touch
---


***This is an update of the [initial post]({% post_url 2015-02-15-ubuntu-touch-app-wishlist %}), contains much more links and all the links have been changed to the new [uApp Explorer][uapp-explorer].***

Basic functionality is there, the most important APIs have been implemented, the first couple thousand phones have been shipped to their buyers and we got some shiny apps - but most of them are webapps. Time to extend the app store with more native apps!

Over the last couple of months I compiled the following wishlist. An entry on this list does not necessarily mean that I would use the app myself, it just means that I have identified a need for it. If an app was part of an older iteration of the wish list and has since then been implemented, I keep the entry to highlight the achievement.

If you are looking for an app idea to implement or are looking for a developer who you can nag into porting his app to Ubuntu Touch, please consider this list.



## Clones of small utilities

- [Car Finder][android-carfinder]

- Bubble level (there is [Level][level-appexplorer])

- Metal detector (...if [this][android-metal] can be taken seriously)

- [Sound Meter][android-soundmeter] (with calibration profiles for known devices)

- Torch (there is [uTorch][utorch-appexplorer])



## Clones/Ports of classic games

|---
| Name | Project link | uApp Explorer | Android Store |
|-|:-:|:-:|:-:|
| 4 in a row/Connect Four/etc. | | | [Connect Four][android-connectfour]
| Backgammon | | | [Backgammon Plus][android-backgammonplus]
| Battleships | | | [Sea Battle][android-seabattle]
| Bindoku (Binary Sudoku) | | | [Link][bindoku]
| Breakout | | | [ArkaBall free][android-arkaball]
| Card games | [Euchre][euchre-launchpad], [Pairs][pairs-launchpad] | [Euchre][euchre], [Pairs][pairs-appexplorer], [Blackjack][blackjack-appexplorer]
| Checkers and variants (Chinese Checkers etc.) | [Checkers][launchpad-checkers]
| Circle the Dot | | | [Circle the Dot][android-circlethedot]
| Dice | [Dice Roller][dice-roller-launchpad] | [Dice Roller][dice-roller-appexplorer]
| Frogger | [Frogger for SailfoshOS][frogger-github] | | [Crossy Road][android-crossyroad]
| [Gorillas][gorillas] | | | [Gorillas][android-gorillas]
| Lasers | | | [Lazors][android-lazors]
| Mastermind | [Mastermind][launchpad-mastermind]
| Math games (basic algebra etc.) | | | [Math Workout][android-mathsworkout]
| Memory | [Same Cards][samecards-launchpad], [Memory][launchpad-memories-touch] | [Same Cards][samecards-appexplorer]
| Minesweeper | [Minesweeper][launchpad-minesweeper]
| Monopoly | | | [Rento Online][android-monopoly]
| Risk | | | [Rise Wars][android-risewars]
| [Simon Tatham's Portable Puzzle Collection][tatham]
| Same Game | [Same Game][samegame-launchpad] | [Same Game][samegame]
| Skyward | | | [Skyward][android-skyward]
| Snake | | [Snake][snake-appexplorer]
| Space Defender | | | [Space Intruders][android-spaceintruders]
| Tic-Tac-Toe | [Noughts-and-Crosses][noughts-and-crosses] | [zTTT][zttt-appexplorer]


* Trivial Pursuit.  There was a digital version of the game once, with downloadable trivia question packages, maybe the file format could be reverse-engineered and the files loaded if the user provides them?




## Clones of popular apps

I think we need clones of these because Ubuntu Touch is too different from other platforms for a direct port.

- AdBlock/AdFree

- Audio/Video format converter

- Audio Memo Recorder. Maybe this could be integrated in the Notes app somehow?

- Backup. This is extremely important, phones store a lot of data and most users will get a new phone within less than three years, so backup and restore should be as painless as possible. It's a horror on Android, let's do better.

- Barcode scanner (***Update:*** [Tagger][tagger-appexplorer] has been in the store for a long time.)

- Business card scanner

- A Camera app on par with the Google Camera App. I use the Panorama and Photo Sphere functions quite often.

- Native clients for all major cloud file sync services (Syncthing, ownCloud, DropBox, MEGA etc.)

- DLNA client to stream audio/video from the local network. This should probably be built into the Media Player?

- DLNA server to stream audio/video to the local network.

- FTP/SCP/SFTP/SMB file transfer client, maybe built into the file manager? And with Content Hub support? (***Update: The File Manager is apparently getting some SMB support.***)

- FM Radio

- [Folder size][foldersize]

- Desktop integration like AirDroid. Transfer files between desktop and phone, show phone notifications on the desktop, write SMS from the desktop etc.

- [TCP/IP network scanner]({% post_url 2015-01-17-unprivileged-icmp-sockets-on-linux %}) (as soon as the situation regarding unprivileged ICMP sockets has been resolved)

- [WiFi scanner]({% post_url 2015-01-30-extending-the-ubuntu-touch-connectivity-api-for-wifi-scanning %}) (as soon as the connectivity-api exposes the necessary functionality)

- [BlueTooth scanner]({% post_url 2015-01-31-extending-the-ubuntu-touch-connectivity-api-for-bluetooth-scanning %}) (as soon as the connectivity-api exposes the necessary functionality)

- [Mobile network scanner]({% post_url 2015-01-31-extending-the-ubuntu-touch-connectivity-api-for-mobile-network-scanning %}) (as soon as the connectivity-api exposes the necessary functionality)
- Simple video editor

- Task automation like [Tasker][tasker]

- Web Radio client (***Update:*** There are some, like [uRadio][uradio-appexpplorer].)





## Clones/Ports of useful single-purpose database apps

These apps could probably be ported, but they are so simple that they can be easily cloned.

- [Dangerous goods][gefahrguthelfer]

- [License plates][kennzeichen]

- Country and State Capitals





## Ports of popular, commercial games

I think some of them already have an HTML5 version, so they could possibly be ported as fast as Cut the Rope.

- Angry Birds (***Update:*** There is a [webapp][angrybirds-appexplorer])

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

- [Aard Dictionary][aard]. Aard allows you to read offline versions of many dictionaries and databases, e.g. Wikipedia, Wikivoyage, Wikiquote etc. I use it extensively when traveling. There are several open-source implementations of the libraries and frontends for different platforms, e.g. Android and Desktop Linux. (***Update:*** I am [working on it][sturmflut-cslob], albeit slowly.)

- Comic Book Reader

- Firefox

- Native IRC client

- [OsmAnd][osmand] or any other turn-by-turn navigation app with offline maps support, preferably with OpenStreetMap support. I use it extensively, not just abroad. (***Update:*** the bq Aquaris E4.5 Ubuntu Edition comes with HERE Maps.)

- Password Safes. I don't use them and don't like them, but other people do.

- Remote frontends for Torrent clients (Deluge etc.)

- Scuba Diving logger and gas calculator. There are a lot of divers out there, including Torvalds himself. (***Update:*** There are now two, [Scuba Gas Manager][scubagasmanager-appexplorer] and [VooDoo Gas][voodoogas-appexplorer]!)

- Tor

- Vector graphics editor like SketchBook Express.

- [VideoLAN Client (VLC)][videolan] (***Update:*** There is a [Remote Control for VLC][vlc-control].)





## Ports of popular closed source apps

The source code for the following apps is not available, and they often rely on proprietary APIs, so we probably have to nag the authors.

- Booking.com, Airbnb, Kayak, EBookers and friends. There are webapps, but native apps come with useful features like offline support.

- Carsharing apps. A webapp may be enough, but I am not sure as I don't use Carsharing.

- Cleverbot (***Update:*** There is [Kirino][kirino-appexplorer].)

- Delivery trackers for all major delivery services (***Update:*** There is [Delivery Tracker][deliverytracker-appexplorer].);

- E-Learning for languages (Babbel, busuu, duolingo etc.) and other areas. (***Update:*** There is [Lang][lang-appexplorer].)

- Fitness tracker apps like FitBit, FitApp, Runtastic etc. (There is [uFit][ufit] in the store, and there's an initial version of [Trailblazer Workout Tracker][trailblazer].)

- FlightRadar24

- Flipboard (***Update:*** There is a [webapp][flipboard-appexplorer].)

- Gas price comparison (***Update:*** There is [Tanken][tanken-appexplorer].)

- Google Earth

- Google Goggles

- Google Hangouts or a replacement. WebRTC might be an option.

- Google Sky Map (***Update:*** There is [Réaltaí][realtai-appexplorer].)

- Instagram (***Update:*** There is a [webapp][instagram-appexplorer].)

- Flickr (***Update:*** There are now two webapps, a [scope][flickr-scope-appexplorer] and even an [uploader][flickr-uploader-appexplorer].)

- Imgur (***Update***: There are now two webapps, a [native app][imgur-appexplorer], a [Content Hub plugin][imgur-share-appexplorer] and a [scope][imgur-scope-appexplorer])

- Snapchat/Slingshot/etc.

- Dictionary/Language Translator with offline support. (***Update:*** [Known Dictionary][known-dictionary-appexplorer] seems to do the job.)

- LinkedIn. There is a webapp, but it lacks platform integration.

- Native Facebook and Facebook Messenger apps. I don't like Facebook, but people are going to want it, and the webapp is powerful, but lacks platform integration.

- An Office Suite, preferably LibreOffice.

- Secure Online Banking for multiple accounts.

- Personal finance management (***Update:*** There is [Chancho][chancho-appexplorer]

- [Prey][prey] or some other anti-theft system.

- Public transportation apps like [DB Navigator][db-navigator] or [Handyticket.de][handyticket]. [fahrplan][fahrplan] already implements many things, but you can't e.g. buy tickets with it, which is one of my major use cases for a smartphone. At least Deutsche Bahn was well-known for also releasing their apps for more "exotic" platforms like Palm handhelds or Symbian devices, and lots of functionality in the Android app is just implemented as a web views, so there is a slight possibility for a somewhat native UT app.

- Re-commerce apps like Re-Buy, Momox, etc. I think something like [Werzahltmehr][android-werzahltmehr] ("Who pays more?") could be built from [Tagger][tagger-appexplorer].

- Shazam, SoundHound or any other song recognition service. (There is [Eyrie][eyrie].)

- Skobbler. We will never get a native, full-featured Google Maps app anyway, and Skobbler might be a worthy replacement.

- Skype or Viber or LINE or Tango or Wire or a replacement, but the replacement will have to be *very* good if it wants to successfully compete against Skype.

- [Speedtest.net][speedtest]

- StoryClash

- TripAdvisor: There is a webapp, but the native app can download content for offline use.

- Uber/MyTaxi/etc.

- WeChat for chinese users.

- WerStreamt.es? ("Who streams it?"), tells you which video streaming provider offers your favourite content.

- WhatsApp. I don't like it, and most smartphones have the power to run both WhatsApp and Telegram at the same time, so you can probably talk most of your friends into installing Telegram, but when Ubuntu Touch devices become generally available the lack of a WhatsApp client will become problematic. And no, WhatsApp Web is not a solution.

- Wikitude

- Xing. There is a webapp, but it lacks platform integration.

- Zedge





## Ports of popular open source libraries and engines

- [Kivy][kivy]

- libSDL. This is [in the works][libsdl-github], albeit slowly.

- [LÖVE][love2d]



[uapp-explorer]: https://uappexplorer.com/


[android-carfinder]: https://play.google.com/store/apps/details?id=com.nomadrobot.mycarlocatorfree

[utorch-appexplorer]: https://uappexplorer.com/app/com.ubuntu.developer.majster-pl.utorch

[level-appexplorer]: https://uappexplorer.com/app/level.mhall119

[android-metal]: https://play.google.com/store/apps/details?id=kr.sira.metal

[android-soundmeter]: https://play.google.com/store/apps/details?id=kr.sira.sound

[android-connectfour]: https://play.google.com/store/apps/details?id=air.com.jogatina.connectfour
[android-backgammonplus]: https://play.google.com/store/apps/details?id=net.peakgames.mobile.android.tavlaplus.android
[android-seabattle]: https://play.google.com/store/apps/details?id=com.byril.seabattle
[android-arkaball]: https://play.google.com/store/apps/details?id=prog.ball
[android-circlethedot]: https://play.google.com/store/apps/details?id=com.ketchapp.circlethedot
[android-crossyroad]: https://play.google.com/store/apps/details?id=com.yodo1.crossyroad
[android-gorillas]: https://play.google.com/store/apps/details?id=de.ptkapps.gorillas.android
[android-lazors]: https://play.google.com/store/apps/details?id=net.pyrosphere.lazors
[android-mathsworkout]: https://play.google.com/store/apps/details?id=com.akbur.mathsworkout
[android-monopoly]: https://play.google.com/store/apps/details?id=air.bg.lan.Monopoli
[android-risewars]: https://play.google.com/store/apps/details?id=com.allso.risewarsads
[android-skyward]: https://play.google.com/store/apps/details?id=com.ketchapp.skyward
[android-spaceintruders]: https://play.google.com/store/apps/details?id=game.spaceintruders

[app-collection]: https://wiki.ubuntu.com/Touch/Collection

[tasker]: https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm

[bindoku]: https://play.google.com/store/apps/details?id=de.lochmann.bindoku

[pairs-launchpad]: https://launchpad.net/pairs-game
[pairs-appexplorer]: https://uappexplorer.com/app/com.ubuntu.developer.robert-ancell.pairs
[euchre-launchpad]: https://launchpad.net/euchre
[euchre]: https://uappexplorer.com/app/com.ubuntu.developer.robert-ancell.euchre

[dice-roller-launchpad]: https://launchpad.net/dice-roller
[dice-roller-appexplorer]: https://uappexplorer.com/app/com.ubuntu.developer.robert-ancell.dice-roller

[frogger-github]: https://github.com/Hootee/frogger

[blackjack-appexplorer]: https://uappexplorer.com/app/com.wellsb.blackjack-app

[launchpad-checkers]: https://launchpad.net/checkers

[gorillas]: http://en.wikipedia.org/wiki/Gorillas_%28video_game%29

[launchpad-mastermind]: https://code.launchpad.net/~hansueli-burri/+junk/Mastermind

[samecards-launchpad]: https://launchpad.net/samecards
[samecards-appexplorer]: https://uappexplorer.com/app/samecards.seb128
[launchpad-memories-touch]: https://code.launchpad.net/~sebastijan-kuzner/memories-touch/trunk

[launchpad-minesweeper]: https://launchpad.net/minesweeper-touch

[tatham]: http://www.chiark.greenend.org.uk/~sgtatham/puzzles/

[samegame-launchpad]: https://launchpad.net/samegame
[samegame]: https://uappexplorer.com/app/com.ubuntu.developer.ken-vandine.samegame

[snake-appexplorer]: https://uappexplorer.com/app/com.wellsb.snake-port

[noughts-and-crosses]: https://github.com/jamesodhunt/qml-noughts-and-crosses
[zttt-appexplorer]: https://uappexplorer.com/app/com.ubuntu.developer.roman2861.zttt

[uradio-appexpplorer]: https://uappexplorer.com/app/uradio.rubenxparra

[foldersize]: https://play.google.com/store/apps/details?id=com.foldersize

[gefahrguthelfer]: https://play.google.com/store/apps/details?id=at.knorre.dangerousgoods

[kennzeichen]: https://play.google.com/store/apps/details?id=flaotec.KFZ_Kennzeichen

[angrybirds-appexplorer]: https://uappexplorer.com/app/angrybirdschrome.markcortbass

[machines-vs-machines]: https://uappexplorer.com/app/com.ubuntu.developer.mzanetti.machines-vs-machines

[ufit]: https://uappexplorer.com/app/com.ubuntu.developer.mzanetti.ubuntu-fitbit-app

[trailblazer]: https://code.launchpad.net/~marin-bareta/trailblazer/main

[prey]: https://preyproject.com/

[aard]: http://aarddict.org/

[db-navigator]: http://www.bahn.de/p/view/buchung/mobil/db-navigator.shtml

[handyticket]: https://www.handyticket.de/

[fahrplan]: https://uappexplorer.com/app/com.ubuntu.developer.mzanetti.fahrplan2

[sturmflut-cslob]: https://github.com/Sturmflut/cslob

[osmand]: http://osmand.net/

[scubagasmanager-appexplorer]: https://uappexplorer.com/app/com.andydragon.scubagasmanager
[voodoogas-appexplorer]: https://uappexplorer.com/app/com.andydragon.voodoogas

[vlc-control]: https://uappexplorer.com/app/com.ubuntu.developer.jdorleans.vlc-control

[tagger-appexplorer]: https://uappexplorer.com/app/com.ubuntu.developer.mzanetti.tagger

[kirino-appexplorer]: https://uappexplorer.com/app/kirino.hyuchia

[deliverytracker-appexplorer]: https://uappexplorer.com/app/deliverytracker.randy-jr-olive

[lang-appexplorer]: https://uappexplorer.com/app/com.ubuntu.developer.boghison.lang

[flipboard-appexplorer]: https://uappexplorer.com/app/flipboardcom-ldp.alci

[tanken-appexplorer]: https://uappexplorer.com/app/com.ubuntu.developer.derjasper.tanken

[realtai-appexplorer]: https://uappexplorer.com/app/com.ubuntu.developer.doflah.realtai

[instagram-appexplorer]: https://uappexplorer.com/app/instagram.fournux

[flickr-scope-appexplorer]: https://uappexplorer.com/app/com.canonical.scopes.flickr
[flickr-uploader-appexplorer]: https://uappexplorer.com/app/it.mardy.uploader

[imgur-appexplorer]: https://uappexplorer.com/app/imgur.roman2861
[imgur-share-appexplorer]: https://uappexplorer.com/app/imgur.mzanetti
[imgur-scope-appexplorer]: https://uappexplorer.com/app/imgur.canonicalpartners

[chancho-appexplorer]: https://uappexplorer.com/app/chancho.mandel

[known-dictionary-appexplorer]: https://uappexplorer.com/app/knowndict.benyfu

[android-werzahltmehr]: https://play.google.com/store/search?q=werzahltmehr&c=apps

[eyrie]: https://uappexplorer.com/app/com.mikeasoft.eyrie

[speedtest]: http://www.speedtest.net/

[videolan]: http://www.videolan.org/vlc/


[kivy]: http://kivy.org/

[libsdl-github]: https://github.com/Sturmflut/ubuntu-touch-sdl-template

[love2d]: https://love2d.org/

